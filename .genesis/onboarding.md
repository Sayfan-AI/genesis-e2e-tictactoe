# Onboarding: genesis-e2e-tictactoe

## Goal

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




## Instructions

This is the onboarding issue. The onboarding agent should:

1. Review the goal above and ask the human clarifying questions
2. Break the goal down into high-level milestones with done criteria
3. Only detail the first milestone — future milestones stay high-level
4. Begin execution on milestone 1
5. Close this issue when onboarding is complete and milestone 1 is underway
