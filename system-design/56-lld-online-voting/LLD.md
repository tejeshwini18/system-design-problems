# LLD: Online Voting System

## 1. Requirements

- **Election:** id, title, start/end time, **candidates** (or options); **voters** (eligible users).
- **Vote:** One vote per voter per election (or per question in a poll); **anonymous** or attributed (per policy); **immutable** (no edit/delete after cast).
- **Result:** Count votes per candidate; result visible after end time (or live tally).
- **Integrity:** Prevent double voting; optional **audit** (proof of count without revealing who voted whom).
- **Optional:** Multiple questions per election; ranked choice; eligibility (by group/role).

---

## 2. Core Classes

```text
Election – id, title, startTime, endTime, status (UPCOMING/ACTIVE/ENDED)
Candidate – id, electionId, name
Voter – userId, eligibleElectionIds (or check via group)
Vote – electionId, candidateId (or optionId), userId?, castAt   // userId null if anonymous
VotingService – castVote(electionId, candidateId, userId) → success/duplicate/not_eligible/closed
  getResults(electionId) → Map<candidateId, count>   // only if ended or policy allows
  hasVoted(electionId, userId) → boolean   // for "already voted" check
ElectionService – createElection(...), getEligibleVoters(electionId)
```

---

## 3. Double-Vote Prevention

- **DB:** UNIQUE(electionId, userId) on Vote table; if userId is stored. On INSERT, duplicate key → return "already voted."
- **Anonymous:** Store hash(userId + electionId) or token issued after auth; one token per user per election; vote stores token only; uniqueness on token so one vote per token.
- **Idempotency:** If client retries, same request (electionId, userId) returns same result (already voted) without inserting again.

---

## 4. Results

- **Query:** SELECT candidate_id, COUNT(*) FROM votes WHERE election_id = ? GROUP BY candidate_id.
- **When:** Only after endTime (or if policy allows live); enforce in getResults().
- **Optional:** Publish event on vote (async) to update read model (counts) for fast read; or compute on demand from votes table.

---

## 5. Key Methods

```text
VotingService.castVote(electionId, candidateId, userId) → success | error
VotingService.getResults(electionId) → Map<candidateId, count>
VotingService.hasVoted(electionId, userId) → boolean
ElectionService.createElection(title, start, end, candidateNames)
```
