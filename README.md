<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Expandable Tic Tac Toe AI</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background-color: #f0f8ff;
    }

    h1 {
      margin-top: 20px;
    }

    #game {
      display: grid;
      margin: 20px auto;
      gap: 4px;
      max-width: 90vmin;
    }

    .cell {
      background: white;
      border: 2px solid #1e90ff;
      font-size: 24px;
      aspect-ratio: 1;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
    }

    #status {
      font-size: 20px;
      margin: 15px;
    }

    button {
      padding: 10px 20px;
      font-size: 16px;
      margin-bottom: 20px;
    }
  </style>
</head>
<body>
  <h1>Tic Tac Toe vs AI 🤖</h1>
  <div id="status">Your Turn (X)</div>
  <div id="game"></div>
  <button onclick="resetGame()">Restart Game</button>

  <script>
    let boardSize = 3; // Pehle board 3x3 hoga
    let winLength = 3; // 3 same symbols ki line se win milega
    let board = []; // Board ke cells ko yeh array store karega
    let currentPlayer = "X"; // Game X se start hoga
    let gameActive = true;

    const game = document.getElementById("game");
    const statusText = document.getElementById("status");

    // Board banaye ya expand kare (agar expand = true ho)
    function initBoard(expand = false) {
      if (expand) {
        const newSize = boardSize + 2; // New board size 2 aur badhega
        const newBoard = Array(newSize * newSize).fill("");

        for (let r = 0; r < boardSize; r++) {
          for (let c = 0; c < boardSize; c++) {
            newBoard[(r + 1) * newSize + (c + 1)] = board[r * boardSize + c]; // Old board ko center mein copy karenge
          }
        }

        boardSize = newSize; // Board size update
        winLength += 1; // Win ke liye +1 length chahiye ab
        board = newBoard; // New board assign
      } else {
        board = Array(boardSize * boardSize).fill(""); // Fresh empty board
      }

      renderBoard(); // Board show karna
    }

    // Board ko show karna screen pe
    function renderBoard() {
      game.innerHTML = "";
      game.style.gridTemplateColumns = `repeat(${boardSize}, 1fr)`; // Grid columns set

      board.forEach((cell, index) => {
        const div = document.createElement("div");
        div.className = "cell";
        div.textContent = cell;
        div.addEventListener("click", () => playerMove(index));
        game.appendChild(div);
      });
    }

    // Player ke click se move handle karna
    function playerMove(index) {
      if (!gameActive || board[index]) return;

      board[index] = currentPlayer;
      renderBoard();

      if (checkWin(currentPlayer)) {
        statusText.textContent = `${currentPlayer} wins! 🎉`;
        gameActive = false;
      } else if (isDraw()) {
        expandGame(); // Agar draw hai to board expand karo
      } else {
        currentPlayer = currentPlayer === "X" ? "O" : "X"; // Turn change karo
        statusText.textContent = `${currentPlayer}'s turn`;
        if (currentPlayer === "O") setTimeout(superAiMove, 300); // Agar AI ki turn hai to move karao
      }
    }

    // AI ka move
    function superAiMove() {
      let bestScore = -Infinity;
      let move;

      for (let i = 0; i < board.length; i++) {
        if (board[i] === "") {
          board[i] = "O";
          let score = minimax(board, 0, false);
          board[i] = "";
          if (score > bestScore) {
            bestScore = score;
            move = i;
          }
        }
      }

      if (move !== undefined) {
        board[move] = "O";
        renderBoard();

        if (checkWin("O")) {
          statusText.textContent = "AI wins! 😈";
          gameActive = false;
        } else if (isDraw()) {
          expandGame(); // Agar phir draw ho gaya
        } else {
          currentPlayer = "X";
          statusText.textContent = "Your turn (X)";
        }
      }
    }

    // Minimax algorithm: AI ka smart decision making
    function minimax(boardState, depth, isMax) {
      if (checkWin("O", boardState)) return 10 - depth;
      if (checkWin("X", boardState)) return depth - 10;
      if (boardState.every(cell => cell !== "")) return 0;

      if (isMax) {
        let best = -Infinity;
        for (let i = 0; i < boardState.length; i++) {
          if (boardState[i] === "") {
            boardState[i] = "O";
            best = Math.max(best, minimax(boardState, depth + 1, false));
            boardState[i] = "";
          }
        }
        return best;
      } else {
        let best = Infinity;
        for (let i = 0; i < boardState.length; i++) {
          if (boardState[i] === "") {
            boardState[i] = "X";
            best = Math.min(best, minimax(boardState, depth + 1, true));
            boardState[i] = "";
          }
        }
        return best;
      }
    }

    // Check draw condition
    function isDraw() {
      return board.every(cell => cell !== "") && !checkWin("X") && !checkWin("O");
    }

    // Board ko expand karo draw ke baad
    function expandGame() {
      statusText.textContent = `Draw! Expanding to ${boardSize + 2}x${boardSize + 2}. Need ${winLength + 1} in a row.`;
      initBoard(true); // Board expand karo
      gameActive = true;
      currentPlayer = "O"; // AI ka turn draw ke baad
      setTimeout(superAiMove, 400); // AI move execute karo
    }

    // Win checker for any size board
    function checkWin(player, b = board) {
      const get = (r, c) => {
        if (r < 0 || c < 0 || r >= boardSize || c >= boardSize) return null;
        return b[r * boardSize + c];
      };

      for (let r = 0; r < boardSize; r++) {
        for (let c = 0; c < boardSize; c++) {
          if (
            checkLine(r, c, 1, 0, player, get) || // Horizontal
            checkLine(r, c, 0, 1, player, get) || // Vertical
            checkLine(r, c, 1, 1, player, get) || // Diagonal \
            checkLine(r, c, 1, -1, player, get)   // Diagonal /
          ) return true;
        }
      }
      return false;
    }

    // Win ke liye continuous symbols check karna
    function checkLine(r, c, dr, dc, player, get) {
      for (let i = 0; i < winLength; i++) {
        if (get(r + dr * i, c + dc * i) !== player) return false;
      }
      return true;
    }

    // Game reset karna
    function resetGame() {
      boardSize = 3;
      winLength = 3;
      currentPlayer = "X";
      gameActive = true;
      initBoard(false); // Fresh board
      statusText.textContent = "Your Turn (X)";
    }

    initBoard(false); // Game start karte hi board ban jaye
  </script>
</body>
</html>
