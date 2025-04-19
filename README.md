<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>4096</title>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      background: #faf8ef;
      text-align: center;
      margin: 0;
      padding: 0;
    }
    h1 {
      margin-top: 20px;
    }
    #scoreboard {
      margin: 10px;
      font-size: 18px;
    }
    #board {
      display: grid;
      gap: 5px;
      justify-content: center;
      margin: 10px auto;
    }
    .cell {
      width: 60px;
      height: 60px;
      background: #cdc1b4;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: bold;
      font-size: 20px;
      border-radius: 6px;
      color: #776e65;
    }
    .controls {
      margin: 10px;
    }
    .controls button {
      padding: 10px 20px;
      font-size: 16px;
      margin: 5px;
    }
  </style>
</head>
<body>
  <h1>4096</h1>
  <div id="scoreboard">
    分数: <span id="score">0</span> |
    历史最高: <span id="best">0</span>
  </div>
  <div id="board"></div>

  <div class="controls">
    <button onclick="restartSame()">重新开始</button>
    <button onclick="startNew()">新局</button>
  </div>

  <script>
    const SIZE = 10;
    let board = [];
    let prevBoard = [];
    let score = 0;
    let best = localStorage.getItem('best4096') || 0;

    function getColor(val) {
      const colors = {
        0: "#cdc1b4", 2: "#eee4da", 4: "#ede0c8", 8: "#f2b179",
        16: "#f59563", 32: "#f67c5f", 64: "#f65e3b", 128: "#edcf72",
        256: "#edcc61", 512: "#edc850", 1024: "#edc53f",
        2048: "#edc22e", 4096: "#3c3a32"
      };
      return colors[val] || "#3c3a32";
    }

    function createBoard() {
      board = Array.from({ length: SIZE }, () => Array(SIZE).fill(0));
      addRandom();
      addRandom();
    }

    function drawBoard() {
      const boardDiv = document.getElementById("board");
      boardDiv.innerHTML = "";
      boardDiv.style.gridTemplateColumns = `repeat(${SIZE}, 60px)`;
      board.forEach(row => {
        row.forEach(cell => {
          const cellDiv = document.createElement("div");
          cellDiv.className = "cell";
          cellDiv.style.background = getColor(cell);
          cellDiv.textContent = cell !== 0 ? cell : "";
          boardDiv.appendChild(cellDiv);
        });
      });
      document.getElementById("score").textContent = score;
      document.getElementById("best").textContent = best;
    }

    function addRandom() {
      let empty = [];
      board.forEach((row, i) =>
        row.forEach((val, j) => {
          if (val === 0) empty.push([i, j]);
        })
      );
      if (empty.length === 0) return;
      let [x, y] = empty[Math.floor(Math.random() * empty.length)];
      board[x][y] = Math.random() < 0.9 ? 2 : 4;
    }

    function rotateLeft() {
      const copy = board.map(row => row.slice());
      for (let i = 0; i < SIZE; i++) {
        for (let j = 0; j < SIZE; j++) {
          copy[SIZE - j - 1][i] = board[i][j];
        }
      }
      board = copy;
    }

    function rotateRight() {
      const copy = board.map(row => row.slice());
      for (let i = 0; i < SIZE; i++) {
        for (let j = 0; j < SIZE; j++) {
          copy[j][SIZE - i - 1] = board[i][j];
        }
      }
      board = copy;
    }

    function compress(row) {
      let newRow = row.filter(val => val);
      for (let i = 0; i < newRow.length - 1; i++) {
        if (newRow[i] === newRow[i + 1]) {
          newRow[i] *= 2;
          score += newRow[i];
          newRow[i + 1] = 0;
        }
      }
      return newRow.filter(val => val).concat(Array(SIZE).fill(0)).slice(0, SIZE);
    }

    function move(dir) {
      saveBoard();
      if (dir === "up") rotateLeft();
      if (dir === "down") rotateRight();
      if (dir === "right") board = board.map(row => row.reverse());

      board = board.map(row => compress(row));

      if (dir === "up") rotateRight();
      if (dir === "down") rotateLeft();
      if (dir === "right") board = board.map(row => row.reverse());
      addRandom();
      drawBoard();
      checkGameOver();
    }

    function saveBoard() {
      prevBoard = board.map(row => row.slice());
    }

    function restartSame() {
      board = prevBoard.map(row => row.slice());
      score = 0;
      drawBoard();
    }

    function startNew() {
      score = 0;
      createBoard();
      drawBoard();
    }

    function checkGameOver() {
      if (board.flat().includes(0)) return;
      for (let i = 0; i < SIZE; i++) {
        for (let j = 0; j < SIZE - 1; j++) {
          if (board[i][j] === board[i][j + 1] || board[j][i] === board[j + 1][i]) return;
        }
      }
      setTimeout(() => {
        let message = "";
        if (score > best) {
          localStorage.setItem("best4096", score);
          best = score;
          message = "游戏结束！恭喜你破纪录了！";
        } else {
          const diff = best - score;
          message = `游戏结束！离历史记录还差 ${diff} 分。`;
        }
        if (confirm(message + "\n点击确定重新开始")) {
          startNew();
        }
      }, 100);
    }

    document.addEventListener("keydown", e => {
      switch (e.key) {
        case "ArrowLeft": move("left"); break;
        case "ArrowRight": move("right"); break;
        case "ArrowUp": move("up"); break;
        case "ArrowDown": move("down"); break;
      }
    });

    let startX, startY;
    document.addEventListener("touchstart", e => {
      startX = e.touches[0].clientX;
      startY = e.touches[0].clientY;
    });
    document.addEventListener("touchend", e => {
      let dx = e.changedTouches[0].clientX - startX;
      let dy = e.changedTouches[0].clientY - startY;
      if (Math.abs(dx) > Math.abs(dy)) {
        if (dx > 30) move("right");
        else if (dx < -30) move("left");
      } else {
        if (dy > 30) move("down");
        else if (dy < -30) move("up");
      }
    });

    createBoard();
    drawBoard();
  </script>
</body>
</html>
