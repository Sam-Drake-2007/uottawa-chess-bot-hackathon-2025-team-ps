# uOttawa Computer Chess Hackathon 2025 – Team PS

## Achievements
- First place overall  
- Fastest Checkmate award  

Built for the uOttawa Computer Chess Hackathon 2025, this bot focuses on optimizing search time, improving evaluation accuracy, and implementing dynamic time management. Our main goal was to make the bot as strong and efficient as possible within strict time limits.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Optimizing Move Time](#optimizing-move-time)
  - [Alpha-Beta Pruning](#alpha-beta-pruning)
  - [Move Ordering](#move-ordering)
- [Optimizing Evaluation](#optimizing-evaluation)
  - [Piece-Square Tables (PSTs)](#piece-square-tables-psts)
  - [Endgame PST Adjustments](#endgame-pst-adjustments)
- [Time Management](#time-management)
  - [Bullet Tiebreaker](#bullet-tiebreaker-10-games)
  - [Endgame](#bullet-endgame-logic)
- [Setup & Installation](#setup--installation)
- [Challenges and Lessons Learned](#challenges-and-lessons-learned)
- [Future Improvements](#future-improvements)
- [Summary](#summary)
- [Team & Credits](#team--credits)
- [License & Attribution](#license--attribution)

---

## Project Overview

The default base bot provided by the hackathon organizers performed poorly in competitive time control settings. Its move generation was slow and lacked positional understanding, often resulting in 30+ second moves at depth 4.  

Our project aimed to:
- Greatly reduce move calculation time  
- Improve move quality through positional awareness  
- Implement dynamic time management for different game phases  

---

## Optimizing Move Time

### Alpha-Beta Pruning

The default minimax search analyzed every node in a depth-4 tree, making it extremely inefficient. We introduced alpha-beta pruning, which stops exploring branches that cannot improve the current best outcome.

In simple terms:  
If a branch leads to a worse position than one already found, we stop searching that branch.  

This dramatically reduced the number of evaluated positions, improving move time without sacrificing depth.

### Move Ordering

Alpha-beta pruning’s efficiency depends heavily on move ordering. By sorting moves so that promising ones (like captures or checks) are explored first, pruning becomes more effective.

We implemented a category-based move ordering system prioritizing:
1. Captures (MVV/LVA)  
2. Checks  
3. Promotions  
4. Quiet moves last  

However, we later discovered that in some endgame checkmate scenarios, this caused alpha-beta to prune winning lines that appeared temporarily worse. This led to one instance where the bot drew a game by repetition instead of checkmating, but overall the speed improvements were significant.

## Optimizing Evaluation

### Piece-Square Tables (PSTs)

Originally, the bot only evaluated material balance, leading to random quiet moves when no captures were available. We integrated Piece-Square Tables (PSTs), assigning each piece a score based on its position on the board.

This allowed the bot to play positional, non-capture moves that improved control and activity rather than just trading material.

### Endgame PST Adjustments

We added different PSTs for the beginning/midgame and endgame. Although PSTs could theoretically change with each capture, implementing fully dynamic tables would be too time-consuming. The simplified version—switching between two main tables—provided significant improvement with minimal computation.

---

## Time Management

Efficient time management is essential for building a competitive chess bot. Different search depths require different amounts of computation time, so we need to balance depth and speed.

In a **3+2 (3 minutes + 2 seconds increment)** game, any move that takes under 2 seconds will add time to the clock. To take advantage of that, we measured how long our bot takes to make moves at each search depth and selected the highest depth where the average move time was around **2 seconds**. This became our **default depth**.

If our clock has a lot of time, we want to use that time to make smarter moves at a higher depth. We tested how long a move takes at **default depth + 1** — for us, that was around **40 seconds at depth 5**. This means that if the bot starts a move with only 40 seconds left, it risks timing out. To prevent this, we programmed the bot to **reduce its depth** once the clock drops below a safe threshold.

For example, if the average move at **depth + 1** takes 40 seconds, we lower the search depth to the default level once the clock hits around **1 minute** (40 seconds plus a 20 second buffer). Running at a depth higher than 5 was not worth it for us, since those moves would take far too long.

### Bullet Tiebreaker (1+0 Games)

In tiebreak situations, games are replayed as **1-minute bullet matches**, and the bot code cannot be changed between rounds. This means our time management logic must handle both formats automatically.

At this speed, a move at **depth + 1** would consume two-thirds of the entire clock — clearly unsustainable. Luckily, the **first 5 moves** are taken directly from the **opening book**, meaning they are played instantly. By the time the bot starts calculating its own moves, the clock has already dropped below 1 minute, and it naturally switches to its **default (faster) depth**.

### Bullet Endgame Logic

In chess, **timing out means an automatic loss**, so the bot must always prioritize making *some* move over making the *best* move. In bullet, the time gain is 0, so even running our **default depth** is not sustainable. To solve this, we reduce the search depth dynamically as time runs out:

- **> 20 seconds:** use `default depth - 1`  
- **> 10 seconds:** use `default depth - 2`  
- **> 5 seconds:** use `default depth - 3` (for us, depth 1, the lowest possible depth)

At **depth 1**, the bot moves nearly instantly, making it virtually impossible to lose on time. This adaptive depth system allows the bot to maintain strong play through most of the game, while automatically prioritizing survival in time pressure. Since late-game positions are simpler, lowering depth doesn’t significantly reduce the quality of play — but it prevents catastrophic timeouts.

---

## Setup & Installation

```bash
# 1. Create virtual environment
python -m venv .venv
.venv\Scripts\activate   # (Windows)
# source .venv/bin/activate  # (Mac/Linux)

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure the bot
cp config.yml.default config.yml
# Add your Lichess token and engine path in config.yml

# 4. Run the bot
python lichess-bot.py
```
Requirements:
- Python 3.9+  
- Stockfish or another UCI-compatible chess engine  

---

## Challenges and Lessons Learned

1. **Leave plenty of time for testing.**  
   We were scrambling near the end because we tried implementing something that didn’t work, so we had to revert to an older version.

2. **Move ordering can be a double-edged sword.**  
   While it made our pruning faster, it occasionally caused the bot to miss mate-in-x lines due to aggressive branch cutting.

3. **Dynamic time management was crucial.**  
   Adapting search depth based on clock time was key to handling multiple time controls efficiently and avoiding timeouts.

---

## Future Improvements

- **Quiescence Search:**  
  We attempted to implement quiescence search near the end of the coding phase, but it introduced instability and had to be reverted. In future versions, this would smooth out evaluation in tactical positions by extending searches in nodes with more potential than others (captures, checks, promotions).

- **Dynamic PST Adjustments:**  
  Currently, the bot switches between two PSTs (midgame and endgame). Updating PSTs more dynamically as pieces are captured could yield stronger positional play.

- **Transposition Tables:**  
  Implementing a hash-based transposition table would allow the bot to reuse previously evaluated positions, increasing efficiency and enabling deeper searches within the same time constraints.

- **Improved Checkmate Recognition:**  
  Adjusting evaluation weighting and move ordering in endgames would help prevent rare threefold repetition draws in winning positions.

---

## Summary

This project demonstrates that any chess bot can achieve competitive strength through efficient search and time optimization.  
By applying classical algorithms thoughtfully, our team created a system that was both fast and reliable under hackathon time constraints.

Our main innovations were:

- **Search Efficiency:** Implementing alpha-beta pruning and ordered move exploration to reduce computation without sacrificing decision depth.  
- **Positional Understanding:** Using piece-square tables to guide non-capture moves and enhance positional play.  
- **Dynamic Time Control:** Adapting search depth to available clock time, ensuring consistent performance across standard and bullet formats.  

These improvements allowed our bot to consistently evaluate positions faster and more intelligently than the default base implementation.  
The result was a robust engine capable of winning games both tactically and strategically — ultimately securing **first place overall** and the **Fastest Checkmate award** at the **uOttawa Computer Chess Hackathon 2025**.

This project highlights how well-implemented classical AI principles can still produce competitive results when optimized with practical constraints in mind.

---

## Team & Credits

**Team PS**  
- **Samuel Drake**  ([GitHub](https://github.com/Sam-Drake-2007))  
- **Thomas Pingot** ([GitHub](https://github.com/9leglama))  

Developed for the **uOttawa Computer Chess Hackathon 2025**.  

---

## License & Attribution

This project is licensed under the **GNU Affero General Public License v3 (AGPL-3.0)**.  
It builds on the official  
[uOttawa Computer Chess Hackathon 2025 base framework](https://github.com/uOttawa-Computer-Chess/uottawa-computer-chess-hackathon-2025-team-ps).  

All modifications and improvements made by Team PS are released under the same AGPL license.  
Users are free to run, modify, and redistribute this project under the same terms, provided attribution to the original base repository and license is maintained.
