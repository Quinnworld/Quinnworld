<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>中国国际象棋AI对弈平台</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f9f9f9; padding: 20px; display: flex; flex-direction: column; align-items: center; }
    h1 { color: #333; }
    #board { width: 400px; height: 400px; margin-top: 20px; }
    #status { margin-top: 10px; font-size: 16px; }
    #controls { margin-top: 10px; }
    button { padding: 8px 16px; font-size: 14px; margin-right: 10px; }
  </style>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard.min.css" />
</head>
<body>
  <h1>中国国际象棋AI对弈平台</h1>
  <div id="controls">
    <button id="newGameBtn">新游戏</button>
    <label>AI深度:
      <select id="aiDepth">
        <option value="10">10</option>
        <option value="15" selected>15</option>
        <option value="20">20</option>
      </select>
    </label>
  </div>
  <div id="board"></div>
  <div id="status">点击“新游戏”开始</div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.13.4/chess.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/stockfish@15.1.1/src/stockfish.js"></script>
  <script>
    let board = null;
    let game = new Chess();
    let engine = STOCKFISH();
    let aiDepthSelect = document.getElementById('aiDepth');
    let statusEl = document.getElementById('status');
    let newGameBtn = document.getElementById('newGameBtn');
    let isPlayerTurn = true;

    engine.onmessage = function(event) {
      let line = typeof event === 'string' ? event : event.data;
      if (line.startsWith('bestmove')) {
        let parts = line.split(' ');
        let move = parts[1];
        game.move({ from: move.substr(0,2), to: move.substr(2,2), promotion: 'q' });
        board.position(game.fen());
        updateStatus();
        isPlayerTurn = true;
      }
    };

    function sendToEngine(cmd) {
      engine.postMessage(cmd);
    }

    function aiMove() {
      sendToEngine('position fen ' + game.fen());
      sendToEngine('go depth ' + aiDepthSelect.value);
    }

    function onDrop(source, target) {
      if (!isPlayerTurn) return 'snapback';
      let move = game.move({ from: source, to: target, promotion: 'q' });
      if (move === null) return 'snapback';
      updateStatus();
      isPlayerTurn = false;
      window.setTimeout(aiMove, 200);
    }

    function updateStatus() {
      let status = '';
      if (game.in_checkmate()) {
        status = '将死！游戏结束。';
      } else if (game.in_draw()) {
        status = '和棋。';
      } else {
        status = '轮到 ' + (game.turn() === 'w' ? '白方(你)' : '黑方(AI)') + '。';
        if (game.in_check()) status += ' 将军！';
      }
      statusEl.innerText = status;
    }

    function startGame() {
      game.reset();
      board.start();
      isPlayerTurn = true;
      updateStatus();
      sendToEngine('ucinewgame');
      sendToEngine('isready');
    }

    newGameBtn.addEventListener('click', startGame);

    board = Chessboard('board', {
      draggable: true,
      position: 'start',
      onDrop: onDrop
    });
  </script>
</body>
</html>
