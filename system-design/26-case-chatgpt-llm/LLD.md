# Low-Level Design: How LLMs Like ChatGPT Work (Serving)

## 1. APIs

```http
POST   /v1/chat/completions
  Body: {
    "model": "gpt-4",
    "messages": [ { "role": "user", "content": "..." }, { "role": "assistant", "content": "..." }, { "role": "user", "content": "..." } ],
    "stream": true,
    "max_tokens": 500,
    "temperature": 0.7,
    "stop": ["\n\n"]
  }
  Response (stream=false): { "choices": [ { "message": { "role": "assistant", "content": "..." }, "finish_reason": "stop" } ], "usage": { "prompt_tokens", "completion_tokens" } }
  Response (stream=true): SSE stream of chunks: data: { "choices": [ { "delta": { "content": "Hi" } } ] }
```

---

## 2. Key Classes / Modules

```text
ChatOrchestrator
  - enqueue(request) → request_id
  - selectModel(request) → model_id, tier
  - dispatch(request) → InferenceWorker

Preprocessor
  - buildInput(messages, model_context_limit) → token_ids[], truncated?
  - tokenize(text) → token_ids
  - truncateOrSummarize(messages, max_tokens) → messages

InferenceWorker
  - loadModel(model_id)  // or get from pool
  - prefill(token_ids) → kv_cache, logits
  - decodeStep(kv_cache, prev_token) → next_token, updated_kv_cache
  - sample(logits, temperature, top_p) → token_id

StreamHandler
  - writeToken(token_id); detokenize; apply safety; send SSE chunk
  - finish(usage); send [DONE] or final message
```

---

## 3. Request Lifecycle (Steps)

1. Gateway: validate body; auth; rate limit; forward to orchestrator.
2. Orchestrator: build messages into prompt; preprocess (tokenize, truncate); enqueue with priority/model.
3. Worker: dequeue; prefill (one forward pass); then loop: sample → decode step → stream token (if stream) → until EOS or max_tokens.
4. Stream: each token sent as SSE: `data: {"choices":[{"delta":{"content":"x"}}]}\n\n`.
5. Final: send `data: {"choices":[{"delta":{},"finish_reason":"stop"}],"usage":{...}}\n\n` and close.

---

## 4. Context Window (Pseudocode)

```text
max_tokens = model.context_length  // e.g. 8192
system_tokens = tokenize(system_prompt)
history_tokens = []
for msg in messages[:-1]:
  history_tokens += tokenize(format(msg))
current_tokens = tokenize(messages[-1])
total = system_tokens + history_tokens + current_tokens
if total > max_tokens:
  drop = total - max_tokens
  // drop from history_tokens (oldest first) or summarize
  history_tokens = history_tokens[drop:] or summarize(history_tokens, max_tokens - len(system_tokens) - len(current_tokens))
input_ids = system_tokens + history_tokens + current_tokens
```

---

## 5. Sampling

- **Greedy:** next_token = argmax(logits).
- **Temperature:** logits /= temperature; softmax; sample from distribution. temperature=0 → greedy.
- **Top_p (nucleus):** Sort logits; cumulative softmax; take smallest set that sums to p; renormalize and sample.
- **Stop:** If decoded token sequence ends with any of stop strings (or EOS token id), set finish_reason=stop and break.

---

## 6. Batching (Continuous Batching)

- **Batch:** Set of in-flight requests; each has its own kv_cache and position.
- **Prefill batch:** New requests added; run prefill for all in batch (variable length: pad or pack).
- **Decode batch:** Each step: for each request in batch, run one decode (next token); variable lengths: some requests finish and leave batch; new requests join from prefill.
- **Scheduler:** Add to batch when slot available; remove when EOS or max_tokens; refill from queue.

---

## 7. Error Handling

- **OOM:** Reject request with 507 or fallback to smaller model.
- **Timeout:** Server-side timeout (e.g. 60 s); stop generation and return partial; set finish_reason=length or timeout.
- **Rate limit:** 429 with Retry-After; client retries with backoff.
- **Invalid request:** 400 (e.g. empty messages, unknown model).
