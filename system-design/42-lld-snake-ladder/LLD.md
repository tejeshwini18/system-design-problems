# LLD: Snake and Ladder Game

## 1. Requirements

- **Board:** 100 cells (1–100). **Snakes:** head → tail (go down). **Ladders:** foot → top (go up).
- **Players:** 2–N; take turns. **Dice:** Roll 1–6 (or 1–6 × number of dice).
- **Move:** Current position + dice value = new position; if new position has snake head, move to tail; if ladder foot, move to top; if > 100, stay (or bounce back).
- **Win:** First to reach 100 exactly (if roll would exceed 100, don’t move or require exact roll).
- **Optional:** Multiple dice; rule "roll 6 again"; save/load game.

---

## 2. Core Classes

```text
Cell – position (1–100); optional: snakeOrLadder (type, endPosition)
Board – cells[101]; getNextPosition(position, diceValue) → newPosition (apply snake/ladder)
Dice – roll() → 1..6
Player – id, name, currentPosition
Game – board, players[], currentPlayerIndex, status (PLAYING/WON)
GameController – createGame(players, snakes[], ladders[]), rollDice(gameId) → { newPosition, nextPlayer, won? }
```

---

## 3. Move Logic

1. diceValue = dice.roll().
2. newPos = currentPosition + diceValue.
3. If newPos > 100: newPos = currentPosition (no move), or implement "bounce" (e.g. 100 - (newPos - 100)).
4. If newPos == 100: player wins; game over.
5. Apply snake/ladder: if board.cells[newPos] has snake (head), newPos = tail; if ladder (foot), newPos = top.
6. Set player.currentPosition = newPos; switch to next player; return result.

---

## 4. Board Setup

- **Snakes:** List of (head, tail) where head > tail; e.g. (62, 5).
- **Ladders:** List of (foot, top) where foot < top; e.g. (2, 38).
- **Board:** Map position → endPosition (if snake head or ladder foot); else position stays.

---

## 5. Key Methods

```text
Board.getNextPosition(position, diceValue) → newPosition
Game.rollDice() → { playerId, diceValue, newPosition, isWin, nextPlayerId }
Game.getCurrentPlayer(), getPlayerPosition(playerId)
```
