# LLD: Chess Game

## 1. Requirements

- **Board:** 8×8; **Pieces:** King, Queen, Rook, Bishop, Knight, Pawn (white and black).
- **Turn:** White then Black alternately; one piece moved per turn (or castling as special move).
- **Move:** Validate: piece at source, destination valid for that piece type, path clear (except Knight), no self-check, optional capture.
- **Check:** King under attack; player must get out of check.
- **Checkmate:** King in check with no valid move; game over.
- **Stalemate, draw:** Optional (fifty-move, repetition, insufficient material).

---

## 2. Core Classes

```text
Piece (abstract) – color (WHITE/BLACK), type; canMove(board, from, to) → boolean
  King, Queen, Rook, Bishop, Knight, Pawn extend Piece; each implements move rules

Board – cells[8][8]; getPiece(cell), setPiece(cell, piece), removePiece(cell)
Cell – row, col
Move – fromCell, toCell, piece, capturedPiece?
Game – board, currentTurn (WHITE/BLACK), moveHistory[], status (PLAYING/CHECK/CHECKMATE/STALEMATE)
GameController – createGame(), move(gameId, from, to) → valid/invalid, check/checkmate
```

---

## 3. Move Validation (per piece)

- **King:** One square any direction; castling: king + rook never moved, path clear, not in check.
- **Queen:** Any direction, any distance; path clear.
- **Rook:** Horizontal or vertical; path clear.
- **Bishop:** Diagonal; path clear.
- **Knight:** L-shape (2+1); can jump.
- **Pawn:** Forward 1 (or 2 from start); capture diagonal; en passant optional.

**Common:** from and to valid; piece at from is current player’s; to empty or opponent piece; path clear (except Knight); after move, own king not in check (simulate move, check if king attacked).

---

## 4. Check / Checkmate

- **Check:** After move, opponent’s king is under attack (any opponent piece can capture king’s cell on next move).
- **Checkmate:** Current player’s king in check and no valid move (for any piece) removes check.
- **Implementation:** getAttackingPieces(board, kingCell, opponentColor); getValidMoves(board, currentColor); if king in check and no move leads to safe state → checkmate.

---

## 5. Design Patterns

- **Strategy:** Move strategy per piece type (KingMoveStrategy, etc.) or polymorphic Piece.canMove().
- **State:** Game state (Playing, Check, Checkmate); optional Command pattern for undo (store Move history and reverse).

---

## 6. Key Methods

```text
Board.getPiece(cell), Board.isPathClear(from, to), Board.isAttacked(cell, byColor)
Piece.canMove(board, from, to)
Game.move(from, to) → { success, check, checkmate }
Game.getValidMoves(cell) → List<Cell>
```
