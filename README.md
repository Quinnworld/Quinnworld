<!DOCTYPE html>
<html lang="zh">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>中国国际象棋平台</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f9f9f9;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 20px;
    }
    h1 {
      color: #333;
    }
    #board {
      width: 400px;
      height: 400px;
    }
    #status {
      margin-top: 10px;
      font-size: 16px;
    }
  </style>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard.min.css" />
</head>
<body>
  <h1>中国国际象棋平台</h1>
  <div id="board"></div>
  <div id="status">游戏状态</div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.13.4/chess.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard.min.js"></script>
  <script>
    const game = new Chess();
    const board = Chessboard('board', {
      draggable: true,
      position: 'start',
      onDrop: (source, target) => {
        const move = game.move({
          from: source,
          to: target,
          promotion: 'q' // 兵升变为后
        });

        if (move === null) return 'snapback';
        updateStatus();
      }
    });

    function updateStatus() {
      let status = '';
      if (game.in_checkmate()) {
        status = '将死！游戏结束。';
      } else if (game.in_draw()) {
        status = '和棋。';
      } else {
        status = '轮到 ' + (game.turn() === 'w' ? '白方' : '黑方') + '。';
        if (game.in_check()) {
          status += ' 将军！';
        }
      }
      document.getElementById('status').innerText = status;
    }
  </script>
</body>
</html>
