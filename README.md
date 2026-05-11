# genesis-e2e-tictactoe

Build a two-player tic-tac-toe web app deployed to GitHub Pages.

Requirements:
- A single static page (HTML + CSS + inline or co-located JS) served from GitHub Pages on the `main` branch (either root or `/docs`).
- Two players alternate clicks: X goes first, then O. After each click, check for a win or draw and display the result.
- A "reset" control restores an empty board.

Testability contract (must be honored — an external Playwright e2e test depends on these selectors):
- Each cell is an interactive element (button or div) with attribute `data-cell="N"` where N is 0..8, ordered left-to-right then top-to-bottom.
- After a move, the cell's text content is exactly "X" or "O" (uppercase, no extra whitespace).
- A status element with `id="status"` displays the current game state. Its text contains the word "win" (case-insensitive) when a player wins, "draw" when the game ends in a draw, and neither otherwise.
- A reset element with `id="reset"` clears the board when clicked.

Quality: no frameworks needed, but the code should be clean and tested if practical. Deploy via GitHub Actions or the standard Pages-from-branch mechanism — whichever you prefer.


---

*Bootstrapped by [Genesis](https://github.com/Sayfan-AI/genesis) — an autonomous agentic AI dev system.*
