# LLD: Bowling Alley Machine

## 1. Requirements

- **Game:** 10 **frames**; each frame 1 or 2 rolls (except 10th: 2 or 3 if strike/spare). **Pins:** 10; knock down = score.
- **Scoring:** Frame score = pins knocked + **bonus**: strike (all 10 in one roll) = next 2 rolls added; spare (all 10 in 2 rolls) = next 1 roll added. 10th frame: strike gives 2 extra rolls; spare gives 1 extra roll.
- **Lane:** Multiple lanes; each lane has current **game** (or none); **players** in game (1–4); **current player** and **current frame/roll**.
- **Input:** Roll(pins): add pins to current frame; update score; if frame complete, advance; if game complete, show final score.
- **Optional:** Multiple games (queue); lane assignment; history.

---

## 2. Core Classes

```text
Frame – frameNumber, roll1?, roll2?, roll3? (10th only); getScore() + bonus from next frames
Game – players: List<Player>, currentPlayerIndex, currentFrameIndex; frames: List<Frame> per player (10 each)
  roll(pins): add to current player's current frame; if strike (pins=10) in non-10th, advance frame; if 2 rolls in frame (or spare/strike in 10th), advance; if frame 10 complete for player, advance player; if all players done frame 10, game over
Player – id, name; frames: List<Frame>; getTotalScore() = sum of frame scores (with bonus computed)
Lane – laneId, currentGame?
BowlingService – startGame(laneId, playerIds); roll(laneId, pins); getScore(laneId, playerId); getCurrentPlayer(laneId)
```

---

## 3. Score Calculation

- **Frame 1–9:** score = roll1 + roll2 (or roll1 only if strike). Strike: bonus = next 2 rolls (could be next frame). Spare: bonus = next 1 roll. Compute when those rolls exist (so after later frames filled).
- **Frame 10:** Up to 3 rolls; strike: roll1=10, then 2 more rolls; spare: roll1+roll2=10, then 1 more; else 2 rolls. Score = sum of 3 (or 2) rolls.
- **Total:** Sum of 10 frame scores (each frame score includes bonus when applicable).

---

## 4. Roll Logic

1. Add pins to current player’s current frame (roll1 or roll2 or roll3 for 10th).
2. If strike (pins=10) in frame 1–9: advance to next frame for this player (one roll ends frame).
3. If two rolls in frame (or frame 10 complete): advance frame; if frame > 10, this player done. If all players done (or single player done), game over.
4. If current player done for this round (all rolled for current frame), advance to next player; if all players done frame, advance frame index for all.
- **Simpler:** One “current” state: (playerIndex, frameIndex, rollInFrame). On roll: fill roll; update current (next roll or next frame or next player); recalc scores for frames that now have bonus data.

---

## 5. Key Methods

```text
Game.roll(pins)
Game.getScore(playerId) → total
Game.getCurrentPlayer() → playerId
Game.isOver() → boolean
BowlingService.startGame(laneId, playerIds)
BowlingService.roll(laneId, pins)
```
