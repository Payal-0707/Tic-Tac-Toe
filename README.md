# Tic-Tac-Toe Game Website (C + HTML)

**Course:** 1st Semester BTech CSE – Mini Project
**Languages used:** C (all logic) and HTML (display only)
**No JavaScript, CSS, Python or PHP is used.**

---

## 1. Project Introduction

This project is a two-player Tic-Tac-Toe game that runs in a web browser.
Player X and Player O take turns clicking cells of a 3 × 3 board. The program
detects a win or a draw, keeps a scoreboard, and lets the players restart the
game or reset the scores.

All game rules are written in **C**. The C program also *prints* the HTML page
that the browser shows. A small static `index.html` is the welcome page.

## 2. Objectives

1. Apply basic C concepts: variables, arrays, loops, if-else, functions, strings.
2. Break a problem into small functions (`new_game`, `check_winner`, `is_full`, ...).
3. Understand how a C program can produce a web page (CGI).
4. Build a complete, working project that is easy to explain.

## 3. Files in the Project

```
tictactoe-project/
├── index.html            <- Welcome page (pure HTML)
├── cgi-bin/
│   └── tictactoe.c       <- The whole game (C program)
└── README.md             <- This file
```

## 4. How C is Connected to HTML (CGI) – and Why It Is Needed

**The problem:** A browser can only understand HTML. It cannot run a C program.
A normal `.html` file is also *static*, so it can't decide who won.

**The solution: CGI (Common Gateway Interface).**
CGI is a simple rule that lets a **web server run a program** and send the
program's **printed output** back to the browser.

```
Browser ──(click a cell)──> Web Server ──runs──> tictactoe.cgi (C program)
Browser <──(HTML page)───── Web Server <──printf── tictactoe.cgi
```

So the C program just does two things:

1. **Input:** reads the URL text after `?` using `getenv("QUERY_STRING")`.
2. **Output:** prints `Content-Type: text/html` + a blank line, then the HTML
   page using `printf()`.

**Why is CGI necessary?** Without it, there is no way for a browser to run C code.
CGI is the simplest bridge, and it needs no extra language.

**How is the game remembered? (HTTP is "stateless")**
Every click starts the C program from scratch; it remembers nothing. So the
whole game is stored **inside the link** of each cell:

```
?b=X-O------&t=O&x=1&o=0&d=2&m=4
   │          │   │   │   │   └ m = cell the player clicked (0-8)
   │          │   │   │   └──── d = number of draws
   │          │   │   └──────── o = Player O wins
   │          │   └──────────── x = Player X wins
   │          └──────────────── t = whose turn
   └─────────────────────────── b = board (9 chars: X, O or - for empty)
```

The C program reads this, plays the move, and prints new links for the next click.
No files or database are needed.

## 5. Working Methodology

1. The player opens `index.html` and clicks **START GAME**.
2. The server runs the C program. With no query string, it starts a new game.
3. The program prints the board. Every empty cell is a link `+` that carries the state.
4. The player clicks a cell → the browser asks the server to run the program again
   with the new query string.
5. The program reads the state, places `X`/`O`, checks winner/draw, updates scores,
   and prints the new page.
6. When the game ends, the `+` links disappear and only **Restart Game** and
   **Reset Scores** can be used.

*Restart* = new empty board, **scores kept**. *Reset Scores* = new board + scores set to 0.

## 6. Algorithm

**Board:** array `board[9]`, indexes:

```
 0 | 1 | 2
-----------
 3 | 4 | 5
-----------
 6 | 7 | 8
```

**Main algorithm**

1. START. Print the header `Content-Type: text/html` and a blank line.
2. Set a fresh board (all `-`), `turn = 'X'`, all scores = 0.
3. Read `QUERY_STRING`. If it is empty or invalid, keep the defaults.
   Otherwise load board, turn, scores and clicked cell `move`.
4. **Apply move:**
   1. If `move` is not 0–8, or the cell is not empty, or the game is already over → do nothing.
   2. Put the current player's mark in `board[move]`.
   3. If the player now has 3 in a line → add 1 to that player's score.
   4. Else if the board is full → add 1 to draws.
   5. Else → switch the turn (X↔O).
5. Print the HTML page: status message, 3 × 3 board (two nested loops), scoreboard, buttons.
6. STOP.

**Winner check:** There are 8 winning lines: `{0,1,2} {3,4,5} {6,7,8}` (rows),
`{0,3,6} {1,4,7} {2,5,8}` (columns), `{0,4,8} {2,4,6}` (diagonals).
For each line, if all three cells are equal and not `-`, that mark is the winner.

**Draw check:** If no cell is `-` and there is no winner → draw.

## 7. Simple Flowchart (Text Form)

```
            +---------+
            |  START  |
            +----+----+
                 |
     +-----------v-----------+
     | Print HTML header     |
     +-----------+-----------+
                 |
     +-----------v-----------+
     | Read QUERY_STRING     |
     +-----------+-----------+
                 |
        +--------v--------+  No / invalid
        | Valid state?    +------------------+
        +--------+--------+                  |
                 | Yes                       |
     +-----------v-----------+   +-----------v-----------+
     | Load board,turn,score |   | New board, X's turn,  |
     | and clicked cell      |   | scores = 0            |
     +-----------+-----------+   +-----------+-----------+
                 |                           |
                 +-------------+-------------+
                               |
                  +------------v------------+  No
                  | Move valid & game on?   +----------+
                  +------------+------------+          |
                               | Yes                   |
                  +------------v------------+          |
                  | Place X / O on board    |          |
                  +------------+------------+          |
                               |                       |
                  +------------v------------+  Yes     |
                  | Does player have 3 in   +--> score+1|
                  | a line?                 |          |
                  +------------+------------+          |
                               | No                    |
                  +------------v------------+  Yes     |
                  | Board full?             +--> draws+1|
                  +------------+------------+          |
                               | No                    |
                  +------------v------------+          |
                  | Switch turn (X <-> O)   |          |
                  +------------+------------+          |
                               +-----------+-----------+
                                           |
                              +------------v------------+
                              | Print HTML page:        |
                              | status, board, scores,  |
                              | Restart / Reset links   |
                              +------------+------------+
                                           |
                                      +----v---+
                                      |  STOP  |
                                      +--------+
```

## 8. How to Compile and Run

You need a C compiler (`gcc`) and a web server that supports CGI (Apache).

### Option A – Linux (Ubuntu/Debian) with Apache

```bash
# 1. Install tools
sudo apt update
sudo apt install apache2 gcc

# 2. Enable CGI
sudo a2enmod cgi
sudo systemctl restart apache2

# 3. Compile the C program straight into the CGI folder
sudo gcc cgi-bin/tictactoe.c -o /usr/lib/cgi-bin/tictactoe.cgi
sudo chmod +x /usr/lib/cgi-bin/tictactoe.cgi

# 4. Copy the welcome page
sudo cp index.html /var/www/html/

# 5. Open in browser
#    http://localhost/index.html
```

### Option B – Windows with XAMPP

1. Install **XAMPP** and a C compiler (e.g. MinGW `gcc`). Start **Apache**.
2. Compile:
   ```
   gcc cgi-bin\tictactoe.c -o C:\xampp\cgi-bin\tictactoe.exe
   ```
3. Copy `index.html` to `C:\xampp\htdocs\`.
4. In `index.html` change the link to `cgi-bin/tictactoe.exe`.
5. Open `http://localhost/index.html`.

### Option C – Quick test without a server (great for viva demo)

```bash
gcc cgi-bin/tictactoe.c -o tictactoe.cgi
QUERY_STRING="" ./tictactoe.cgi                                   # first page
QUERY_STRING="b=XX-OO----&t=X&x=0&o=0&d=0&m=2" ./tictactoe.cgi    # X wins
```
You will see the raw HTML printed in the terminal – this is exactly what the
server sends to the browser.

## 9. Sample Test Cases

| Situation | Query string | Result |
|---|---|---|
| First visit | *(empty)* | Empty board, X's turn |
| X completes top row | `b=XX-OO----&t=X&x=0&o=0&d=0&m=2` | "Player X wins!", X score = 1 |
| Last cell, no winner | `b=XOXXOOOX-&t=X&x=0&o=0&d=0&m=8` | "It's a Draw!", Draws = 1 |
| Click on used cell | `b=X--------&t=O&x=0&o=0&d=0&m=0` | Ignored, still O's turn |
| Wrong/edited URL | `b=hack&t=Z` | New game starts |

## 10. Possible Viva Questions

1. **Why CGI?** Browsers can't run C; CGI lets the server run it and return its output.
2. **What is `QUERY_STRING`?** The text after `?` in the URL, given to the program by the server.
3. **Why store the state in the URL?** HTTP is stateless; the program forgets everything after each run.
4. **Why is `board` of size 10?** 9 cells + 1 for the string end character `'\0'`.
5. **How do you find the winner?** Check the 8 possible lines in a 2-D array.
6. **Does refreshing the page give a double score?** No – the link holds the state *before* the move, so repeating it gives the same result.
7. **Difference between Restart and Reset Scores?** Restart keeps scores; Reset sets them to 0.

## 11. Limitations

- Two-player only (no computer opponent).
- Scores are kept in the page link, so they are lost if the player opens `index.html` again.
- The page reloads on every click (no JavaScript is allowed).
- Plain look because CSS is not allowed.
