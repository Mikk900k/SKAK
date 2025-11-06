 (cd "$(git rev-parse --show-toplevel)" && git apply --3way <<'EOF' 
diff --git a/index.html b/index.html
new file mode 100644
index 0000000000000000000000000000000000000000..9e6a58387e30f4f38a8ac95760b583363f0691c6
--- /dev/null
+++ b/index.html
@@ -0,0 +1,618 @@
+<!DOCTYPE html>
+<html lang="da">
+<head>
+  <meta charset="UTF-8" />
+  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
+  <title>Skak med Tips</title>
+  <link rel="preconnect" href="https://fonts.googleapis.com" />
+  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
+  <link
+    href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap"
+    rel="stylesheet"
+  />
+  <style>
+    :root {
+      color-scheme: light dark;
+      --board-size: min(80vw, 640px);
+      --light-square: #f0d9b5;
+      --dark-square: #b58863;
+      --primary-color: #1b4d89;
+      --accent-color: #f7b32b;
+      --background: #f5f5f5;
+      --text: #1c1c1c;
+    }
+
+    * {
+      box-sizing: border-box;
+    }
+
+    body {
+      font-family: 'Roboto', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
+      margin: 0;
+      padding: 0;
+      min-height: 100vh;
+      background: var(--background);
+      color: var(--text);
+      display: flex;
+      justify-content: center;
+      align-items: flex-start;
+      padding: 24px 12px 48px;
+    }
+
+    main {
+      width: 100%;
+      max-width: 960px;
+      background: rgba(255, 255, 255, 0.9);
+      border-radius: 18px;
+      box-shadow: 0 16px 32px rgba(15, 23, 42, 0.12);
+      padding: 32px;
+      display: grid;
+      gap: 32px;
+      grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
+    }
+
+    header {
+      grid-column: 1 / -1;
+      text-align: center;
+    }
+
+    h1 {
+      font-size: clamp(2rem, 2.5vw + 1.2rem, 2.8rem);
+      margin-bottom: 8px;
+      color: var(--primary-color);
+    }
+
+    p.subtitle {
+      margin-top: 0;
+      color: #475569;
+      font-size: 1.05rem;
+    }
+
+    .board-wrapper {
+      display: flex;
+      justify-content: center;
+      align-items: center;
+    }
+
+    #board {
+      display: grid;
+      grid-template-columns: repeat(8, 1fr);
+      border: 8px solid #2f3640;
+      border-radius: 12px;
+      overflow: hidden;
+      width: var(--board-size);
+      height: var(--board-size);
+      background: var(--dark-square);
+      position: relative;
+      box-shadow: inset 0 0 0 1px rgba(0, 0, 0, 0.15);
+    }
+
+    .square {
+      display: flex;
+      justify-content: center;
+      align-items: center;
+      font-size: calc(var(--board-size) / 8 - 8px);
+      line-height: 1;
+      cursor: pointer;
+      user-select: none;
+      transition: background 150ms ease, transform 150ms ease;
+      position: relative;
+    }
+
+    .square.light {
+      background: var(--light-square);
+    }
+
+    .square.dark {
+      background: var(--dark-square);
+    }
+
+    .square.selected::after,
+    .square.hint::after,
+    .square.target::after {
+      content: '';
+      position: absolute;
+      width: 70%;
+      height: 70%;
+      border-radius: 50%;
+      border: 4px solid var(--accent-color);
+      background-color: rgba(247, 179, 43, 0.25);
+    }
+
+    .square.move-option::after {
+      content: '';
+      position: absolute;
+      width: 18%;
+      height: 18%;
+      border-radius: 50%;
+      background: rgba(27, 77, 137, 0.85);
+    }
+
+    .square.capture::after {
+      width: 70%;
+      height: 70%;
+      border-radius: 50%;
+      border: 6px solid rgba(208, 47, 47, 0.85);
+      background: rgba(208, 47, 47, 0.2);
+    }
+
+    .controls {
+      display: flex;
+      flex-direction: column;
+      gap: 16px;
+    }
+
+    .status-card,
+    .hint-card,
+    .history-card {
+      background: #ffffff;
+      border-radius: 16px;
+      padding: 20px;
+      box-shadow: 0 12px 24px rgba(15, 23, 42, 0.08);
+    }
+
+    .status-card h2,
+    .hint-card h2,
+    .history-card h2 {
+      margin-top: 0;
+      color: var(--primary-color);
+      font-size: 1.3rem;
+    }
+
+    .status-text {
+      margin: 0;
+      font-size: 1.05rem;
+    }
+
+    .buttons {
+      display: flex;
+      flex-wrap: wrap;
+      gap: 12px;
+      margin-top: 12px;
+    }
+
+    button {
+      border: none;
+      border-radius: 999px;
+      padding: 12px 22px;
+      font-size: 0.95rem;
+      font-weight: 600;
+      background: var(--primary-color);
+      color: #fff;
+      cursor: pointer;
+      transition: transform 120ms ease, box-shadow 120ms ease, background 120ms ease;
+    }
+
+    button.secondary {
+      background: #94a3b8;
+    }
+
+    button:hover {
+      transform: translateY(-1px);
+      box-shadow: 0 8px 16px rgba(27, 77, 137, 0.2);
+      background: #2361b6;
+    }
+
+    button.secondary:hover {
+      background: #64748b;
+      box-shadow: 0 6px 12px rgba(100, 116, 139, 0.25);
+    }
+
+    button:disabled {
+      opacity: 0.55;
+      cursor: not-allowed;
+      box-shadow: none;
+      transform: none;
+    }
+
+    .hint-output {
+      margin-top: 12px;
+      background: #eef2ff;
+      border-radius: 12px;
+      padding: 14px;
+      font-size: 0.95rem;
+      line-height: 1.5;
+      color: #1e293b;
+    }
+
+    .hint-output strong {
+      color: #0f172a;
+    }
+
+    .history-list {
+      max-height: 320px;
+      overflow-y: auto;
+      margin: 0;
+      padding: 0;
+      list-style: none;
+      display: grid;
+      gap: 8px;
+    }
+
+    .history-list li {
+      background: #f8fafc;
+      border-radius: 10px;
+      padding: 10px 12px;
+      font-size: 0.95rem;
+      display: flex;
+      justify-content: space-between;
+      gap: 16px;
+    }
+
+    @media (max-width: 768px) {
+      main {
+        padding: 20px;
+      }
+
+      #board {
+        width: min(95vw, 480px);
+        height: min(95vw, 480px);
+      }
+
+      .square {
+        font-size: calc(min(95vw, 480px) / 8 - 8px);
+      }
+    }
+  </style>
+</head>
+<body>
+  <main>
+    <header>
+      <h1>Skak med Smarte Tips</h1>
+      <p class="subtitle">
+        Spil klassisk skak direkte i browseren. Klik på en brik for at se lovlige træk,
+        eller få en anbefaling fra vores hurtige tip-maskine.
+      </p>
+    </header>
+
+    <section class="board-wrapper">
+      <div id="board" aria-label="Skakbræt"></div>
+    </section>
+
+    <aside class="controls">
+      <article class="status-card">
+        <h2>Status</h2>
+        <p id="status" class="status-text"></p>
+        <div class="buttons">
+          <button id="hint-btn">Foreslå træk</button>
+          <button id="undo-btn" class="secondary">Fortryd</button>
+          <button id="reset-btn" class="secondary">Nulstil spil</button>
+        </div>
+      </article>
+
+      <article class="hint-card">
+        <h2>Tip &amp; Analyse</h2>
+        <div id="hint-output" class="hint-output">Tryk på "Foreslå træk" for at få et forslag.</div>
+      </article>
+
+      <article class="history-card">
+        <h2>Trækhistorik</h2>
+        <ul id="history" class="history-list"></ul>
+      </article>
+    </aside>
+  </main>
+
+  <script src="https://cdn.jsdelivr.net/npm/chess.js@1.0.0/dist/chess.min.js"></script>
+  <script>
+    const boardElement = document.getElementById('board');
+    const statusElement = document.getElementById('status');
+    const hintOutput = document.getElementById('hint-output');
+    const historyList = document.getElementById('history');
+    const hintBtn = document.getElementById('hint-btn');
+    const undoBtn = document.getElementById('undo-btn');
+    const resetBtn = document.getElementById('reset-btn');
+
+    const pieceUnicode = {
+      pw: '\u2659',
+      pb: '\u265F',
+      nw: '\u2658',
+      nb: '\u265E',
+      bw: '\u2657',
+      bb: '\u265D',
+      rw: '\u2656',
+      rb: '\u265C',
+      qw: '\u2655',
+      qb: '\u265B',
+      kw: '\u2654',
+      kb: '\u265A',
+    };
+
+    const chess = new Chess();
+
+    const pieceNames = {
+      p: 'bonde',
+      n: 'springer',
+      b: 'løber',
+      r: 'tårn',
+      q: 'dame',
+      k: 'konge',
+    };
+
+    const colorNames = {
+      w: 'hvid',
+      b: 'sort',
+    };
+
+    let selectedSquare = null;
+    let availableMoves = [];
+    let highlightedHint = null;
+
+    const files = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'];
+
+    function squareName(fileIndex, rankIndex) {
+      return `${files[fileIndex]}${8 - rankIndex}`;
+    }
+
+    function clearHighlights() {
+      document.querySelectorAll('.square').forEach((square) => {
+        square.classList.remove('selected', 'move-option', 'capture');
+      });
+    }
+
+    function renderBoard() {
+      boardElement.innerHTML = '';
+      const board = chess.board();
+
+      board.forEach((row, rankIndex) => {
+        row.forEach((piece, fileIndex) => {
+          const square = document.createElement('div');
+          square.classList.add('square');
+          square.classList.add((fileIndex + rankIndex) % 2 === 0 ? 'light' : 'dark');
+
+          const squareId = squareName(fileIndex, rankIndex);
+          square.dataset.square = squareId;
+
+          if (piece) {
+            const key = `${piece.type}${piece.color}`;
+            square.textContent = pieceUnicode[key];
+          } else {
+            square.textContent = '';
+          }
+
+          square.addEventListener('click', () => handleSquareClick(squareId));
+
+          boardElement.appendChild(square);
+        });
+      });
+
+      if (highlightedHint) {
+        const from = document.querySelector(`[data-square="${highlightedHint.from}"]`);
+        const to = document.querySelector(`[data-square="${highlightedHint.to}"]`);
+        if (from) from.classList.add('hint');
+        if (to) to.classList.add('target');
+      }
+
+      updateStatus();
+      updateHistory();
+    }
+
+    function handleSquareClick(squareId) {
+      const piece = chess.get(squareId);
+      const isOwnPiece = piece && piece.color === chess.turn();
+
+      if (selectedSquare && availableMoves.some((move) => move.to === squareId)) {
+        const promotionMoves = availableMoves.filter((move) => move.promotion);
+        let promotion = 'q';
+
+        if (promotionMoves.length && promotionMoves.some((move) => move.to === squareId)) {
+          const choice = prompt('Vælg promovering (dame, tårn, løber, springer):', 'dame');
+          const mapping = { dame: 'q', tårn: 'r', toern: 'r', løber: 'b', loeber: 'b', springer: 'n' };
+          promotion = mapping[(choice || '').toLowerCase().trim()] || 'q';
+        }
+
+        chess.move({ from: selectedSquare, to: squareId, promotion });
+        highlightedHint = null;
+        selectedSquare = null;
+        availableMoves = [];
+        clearHighlights();
+        renderBoard();
+        return;
+      }
+
+      if (!isOwnPiece) {
+        selectedSquare = null;
+        availableMoves = [];
+        clearHighlights();
+        return;
+      }
+
+      selectedSquare = squareId;
+      availableMoves = chess.moves({ square: squareId, verbose: true });
+      clearHighlights();
+
+      const fromSquare = document.querySelector(`[data-square="${squareId}"]`);
+      if (fromSquare) fromSquare.classList.add('selected');
+
+      availableMoves.forEach((move) => {
+        const targetSquare = document.querySelector(`[data-square="${move.to}"]`);
+        if (!targetSquare) return;
+        if (move.flags.includes('c') || move.flags.includes('e')) {
+          targetSquare.classList.add('capture');
+        } else {
+          targetSquare.classList.add('move-option');
+        }
+      });
+    }
+
+    function updateStatus() {
+      let statusText = '';
+      const moveColor = chess.turn() === 'w' ? 'Hvid' : 'Sort';
+
+      if (chess.isCheckmate()) {
+        statusText = `Skakmat! ${moveColor === 'Hvid' ? 'Sort' : 'Hvid'} vinder.`;
+      } else if (chess.isDraw()) {
+        statusText = 'Remis. Partiet er uafgjort.';
+      } else {
+        statusText = `${moveColor} er i trækket.`;
+        if (chess.inCheck()) {
+          statusText += ' Pas på: skak!';
+        }
+      }
+
+      statusElement.textContent = statusText;
+      hintBtn.disabled = chess.isGameOver();
+      undoBtn.disabled = chess.history().length === 0;
+    }
+
+    function updateHistory() {
+      const history = chess.history({ verbose: true });
+      historyList.innerHTML = '';
+
+      for (let i = 0; i < history.length; i += 2) {
+        const whiteMove = history[i];
+        const blackMove = history[i + 1];
+
+        const listItem = document.createElement('li');
+        const moveNumber = document.createElement('span');
+        moveNumber.textContent = `${i / 2 + 1}.`;
+
+        const movesText = document.createElement('span');
+        const whiteSan = whiteMove ? whiteMove.san : '';
+        const blackSan = blackMove ? blackMove.san : '';
+        movesText.textContent = `${whiteSan}  ${blackSan}`;
+
+        listItem.appendChild(moveNumber);
+        listItem.appendChild(movesText);
+        historyList.appendChild(listItem);
+      }
+
+      historyList.scrollTop = historyList.scrollHeight;
+    }
+
+    const pieceValues = {
+      p: 100,
+      n: 320,
+      b: 330,
+      r: 500,
+      q: 900,
+      k: 20000,
+    };
+
+    function evaluateBoard(chessInstance) {
+      const board = chessInstance.board();
+      let total = 0;
+
+      board.forEach((row) => {
+        row.forEach((piece) => {
+          if (!piece) return;
+          const value = pieceValues[piece.type];
+          total += piece.color === 'w' ? value : -value;
+        });
+      });
+
+      return total;
+    }
+
+    function minimax(depth, alpha, beta, maximizingPlayer) {
+      if (depth === 0 || chess.isGameOver()) {
+        return evaluateBoard(chess);
+      }
+
+      const moves = chess.moves({ verbose: true });
+
+      if (maximizingPlayer) {
+        let maxEval = -Infinity;
+        for (const move of moves) {
+          chess.move(move);
+          const evalScore = minimax(depth - 1, alpha, beta, chess.turn() === 'w');
+          chess.undo();
+          maxEval = Math.max(maxEval, evalScore);
+          alpha = Math.max(alpha, evalScore);
+          if (beta <= alpha) break;
+        }
+        return maxEval;
+      }
+
+      let minEval = Infinity;
+      for (const move of moves) {
+        chess.move(move);
+        const evalScore = minimax(depth - 1, alpha, beta, chess.turn() === 'w');
+        chess.undo();
+        minEval = Math.min(minEval, evalScore);
+        beta = Math.min(beta, evalScore);
+        if (beta <= alpha) break;
+      }
+      return minEval;
+    }
+
+    function getBestMove(depth = 2) {
+      const moves = chess.moves({ verbose: true });
+      if (!moves.length) {
+        return null;
+      }
+
+      let bestMove = null;
+      let bestEval = chess.turn() === 'w' ? -Infinity : Infinity;
+
+      for (const move of moves) {
+        chess.move(move);
+        const evalScore = minimax(depth - 1, -Infinity, Infinity, chess.turn() === 'w');
+        chess.undo();
+
+        if (chess.turn() === 'w') {
+          if (evalScore > bestEval) {
+            bestEval = evalScore;
+            bestMove = move;
+          }
+        } else {
+          if (evalScore < bestEval) {
+            bestEval = evalScore;
+            bestMove = move;
+          }
+        }
+      }
+
+      return { move: bestMove, evaluation: bestEval / 100 };
+    }
+
+    function formatEvaluation(value) {
+      if (!isFinite(value)) return '0.00';
+      return value.toFixed(2);
+    }
+
+    hintBtn.addEventListener('click', () => {
+      if (chess.isGameOver()) return;
+
+      const result = getBestMove(3);
+      if (!result || !result.move) {
+        hintOutput.textContent = 'Ingen tilgængelige træk.';
+        return;
+      }
+
+      const { move, evaluation } = result;
+      const pieceLabel = pieceNames[move.piece] || move.piece;
+      const colorLabel = colorNames[move.color] || '';
+      const prettyColor = colorLabel ? colorLabel.charAt(0).toUpperCase() + colorLabel.slice(1) : '';
+      hintOutput.innerHTML = `<strong>Forslag:</strong> Flyt ${prettyColor} ${pieceLabel} fra <strong>${move.from}</strong> til <strong>${move.to}</strong>.<br />` +
+        `Forventet stillingsevaluering (positivt = fordel til hvid): <strong>${formatEvaluation(evaluation)}</strong>`;
+
+      highlightedHint = { from: move.from, to: move.to };
+      renderBoard();
+    });
+
+    undoBtn.addEventListener('click', () => {
+      const undone = chess.undo();
+      if (!undone) {
+        hintOutput.textContent = 'Der er ingen træk at fortryde.';
+        return;
+      }
+      highlightedHint = null;
+      clearHighlights();
+      renderBoard();
+      hintOutput.textContent = 'Seneste træk fortrudt. Tryk på "Foreslå træk" for et nyt tip.';
+    });
+
+    resetBtn.addEventListener('click', () => {
+      chess.reset();
+      highlightedHint = null;
+      selectedSquare = null;
+      availableMoves = [];
+      clearHighlights();
+      hintOutput.textContent = 'Spillet er nulstillet. Klar til et nyt parti!';
+      renderBoard();
+    });
+
+    renderBoard();
+  </script>
+</body>
+</html>
 
EOF
)
