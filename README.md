# AI vs AI Chess Arena

Two chess engines play each other in the browser. They start out playing like beginners and get better as the session goes on. You control how fast they play.

It's a single HTML file. No build step, no dependencies, no server. Open it and it runs.

## Run it

Download `ai_vs_ai_chess.html` and open it in any browser.

If you want a live link, turn on GitHub Pages for the repo (Settings > Pages > Source: main branch, root). Rename the file to `index.html` first so the URL is clean:

```
https://your-username.github.io/your-repo/
```

## What it does

- Two engines (White and Black) play full games on their own.
- Skill rises over the session, so the play visibly improves the longer you watch.
- Speed slider from about 0.7 up to 100 moves per second.
- Think-time slider to set how long each engine searches (more time, deeper search, stronger play).
- CPU threads slider that runs the search across multiple cores using Web Workers.
- Play / Pause, Step (one move at a time), New game (keeps the learning), Reset training (back to clueless).
- Last move is highlighted, a king in check glows red, captured pieces show on each name plate.
- Rating bar, tier labels (Beginner up to Grandmaster), and a sparkline of the rating over time.
- Green board styled after the chess.com look.

## How the AI works

The chess engine is written from scratch, no libraries. The move generator was checked with perft, which is the standard way to confirm legal move generation is correct:

- Start position, depth 4: 197,281 nodes (matches the known value).
- Kiwipete position, depth 3: 97,862 nodes (matches the known value).

That covers castling, en passant, promotion, pins, and checkmate.

On top of that runs an alpha-beta search with iterative deepening, a transposition table (Zobrist hashing), quiescence search, and killer-move ordering. The search is bounded by a time budget, so it goes as deep as it can in the time it has instead of using a fixed depth. A single skill value climbs as more moves are played in the session, and it drives three things at once:

- Blunder rate falls from 85% down to near 0. Early on the engines mostly play random legal moves, which is why the opening games look so bad.
- Depth grows with skill and with how much time you give it. It reaches about 4 to 6 ply in the opening and into the teens in simpler endgame positions, because fewer pieces means the search goes much deeper. The depth readout shows what it actually reached on the last move.
- Evaluation shifts from counting material to using piece-square tables plus a positional term.

Use the think-time slider to trade speed for strength. More time means deeper search and better play, but a slower game.

Games restart automatically and the skill carries across them, so leaving it running shows the rating climb and the play tighten up.

## Note on "learning"

This is simulated progression, not real machine learning. The engines follow a skill schedule that deepens their search and lowers their randomness. They do not train a model from game results. It runs instantly in the browser, which is the tradeoff for not doing real self-play training.

## Tech

Plain HTML, CSS, and JavaScript in one file. No frameworks, no external libraries, no internet connection needed once you have the file.

## Using your CPU

The whole thing runs on your machine. There is no server, so all the search work happens on your local CPU.

By default it spreads the search across your cores using Web Workers. It detects how many cores you have and spawns that many workers. Each worker searches a different share of the opening moves (this is called root splitting) and the best result wins. The CPU threads slider lets you pick how many cores to use, and the search-speed readout (in thousands of nodes per second) goes up as you add threads, so you can see the cores working.

The speedup is not perfectly linear, because alpha-beta search does not parallelize cleanly. But it does use the whole chip and search more in the same wall-clock time.

If Web Workers are blocked (some sandboxed previews do this), it falls back to a single thread on the main thread and shows the thread count as `1*`. It still works, it just uses one core and may pause briefly during a long think. On GitHub Pages and when opened as a local file, you get the full multi-core behavior.



The slider sets the scheduled rate. Low-skill games hit the full speed because a blundering engine picks a move instantly. Once skill is high, a single search takes longer, so the real rate caps around 12 to 20 moves per second no matter what the slider says. At very high speeds the slide animation can't keep up and pieces snap to their squares instead of gliding.

## License

MIT. Add a `LICENSE` file if you want one, or change this to whatever you prefer.
