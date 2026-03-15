# LLD: Tic-Tac-Toe Game

## 1. Requirements

- **Board:** 3×3 grid; two players (X and O) take turns.
- **Move:** Player places symbol in empty cell; switch turn; check win/draw.
- **Win:** Same symbol in a row, column, or diagonal.
- **Draw:** No empty cell and no winner.
- **Optional:** N×N board; k-in-a-row to win; replay; undo.

---

## 2. Core Classes

```text
Board – grid[3][3], size; place(row, col, symbol) → success, getWinner() → X | O | null | DRAW, isFull()
Player – id, symbol (X or O)
Game – board, players[2], currentPlayerIndex, status (PLAYING/WON/DRAW)
GameController – createGame(), playMove(gameId, row, col) → result (continue / X wins / O wins / draw)
```

---

## 3. Flow

1. createGame() → Game with empty board; currentPlayer = player0 (X).
2. playMove(gameId, row, col): Validate row,col in range and cell empty; board.place(row, col, currentPlayer.symbol); check board.getWinner() → if X/O wins, set status WON and return; if board.isFull() and no winner, set status DRAW; else switch currentPlayer; return next state.
3. getWinner(): For each row, column, two diagonals, check if all same (and not empty); return that symbol; else if isFull() return DRAW; else null.

---

## 4. Key Methods

```text
Board.place(row, col, symbol) → boolean
Board.getWinner() → X | O | DRAW | null
Board.isFull() → boolean
Game.playMove(row, col) → { status, winner?, message }
```

---

## 5. Design Patterns

- **State:** Game status (Playing, Won, Draw); behavior (playMove allowed only when Playing).
- **Strategy:** Optional different win strategies for N×N and k-in-a-row (inject WinStrategy).

---

## Interview-Readiness Enhancements

### API and consistency
- Mark idempotency requirements for mutation APIs.
- Specify pagination/cursor strategy for list endpoints.
- Clarify consistency guarantees per endpoint/workflow.

### Data model and concurrency
- Explicitly list partition key/index choices and why.
- State optimistic vs pessimistic locking policy and conflict handling.
- Define deduplication/idempotent-consumer strategy for async paths.

### Reliability and operations
- Add explicit failure scenarios with mitigations and degradation behavior.
- Add monitoring/alert thresholds for critical flows and queue lag.
- Document rollout and rollback steps for schema/API changes.

### Validation checklist
- Include unit + integration + load + failure-injection test cases for critical paths.

