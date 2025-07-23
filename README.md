<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Interactive Chess Board</title>
  <!-- Chessboard.js CSS -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard.js/1.0.0/css/chessboard.min.css" integrity="sha512-AfuAx+9e7ZH9u6SjxYVlbqE0VhzQXgCw77L54eOWdM2TB4FKM1xlB2243X1RcGVbX68KVi7HVgxtb4xlzR99PQ==" crossorigin="anonymous" referrerpolicy="no-referrer" />
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      background-color: #f0f0f0;
      height: 100vh;
    }
    h1 {
      margin-top: 20px;
    }
    #board {
      margin-top: 20px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    }
    #status {
      margin-top: 10px;
    }
    #controls {
      margin-top: 15px;
    }
    button {
      padding: 8px 12px;
      border: none;
      border-radius: 4px;
      background-color: #007bff;
      color: white;
      cursor: pointer;
      margin: 0 5px;
    }
    button:disabled {
      background-color: #aaa;
      cursor: not-allowed;
    }
  </style>
</head>
<body>
  <h1>Interactive Chess Board</h1>
  <div id="board" style="width: 480px;"></div>
  <div id="status"></div>
  <div id="controls">
    <button id="flipBtn">Flip Board</button>
    <button id="resetBtn">Reset Game</button>
  </div>

  <!-- Dependencies -->
  <!-- jQuery is required for Chessboard.js -->
  <script src="https://code.jquery.com/jquery-3.6.0.min.js" integrity="sha256-/xUj+3OJ+Y2u7Gv4dX+XrmnH5I2OY/hFsw5J3CEJ5kg=" crossorigin="anonymous"></script>
  <!-- Chess.js: game logic -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/1.0.0/chess.min.js" integrity="sha512-c20vP6BYB0CU2n8A12YQCbRrZ5RG1k0dElREn5zHdipXMMjcEZCIbH8q+OA3bB7DheAO7J26Jb6TSizqeBbEWQ==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
  <!-- Chessboard.js: UI -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard.js/1.0.0/js/chessboard.min.js" integrity="sha512-IuTQ7d2ZJkXgH+lcU+dYZ2R0qq0IvA41ft8ELn3GuTmvKXOY7f+tQFToVak8EQyN+FkW6B/F2mfOlk7bPGpYNQ==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
  <script>
    // Initialization
    var board = null;
    var game = new Chess();

    function onDragStart(source, piece, position, orientation) {
      if (game.game_over() ||
          (game.turn() === 'w' && piece.search(/^b/) !== -1) ||
          (game.turn() === 'b' && piece.search(/^w/) !== -1)) {
        return false;
      }
    }

    function onDrop(source, target) {
      var move = game.move({ from: source, to: target, promotion: 'q' });
      if (move === null) return 'snapback';
      updateStatus();
    }

    function onSnapEnd() {
      board.position(game.fen());
    }

    function updateStatus() {
      var status = '';
      var moveColor = (game.turn() === 'w') ? 'White' : 'Black';

      if (game.in_checkmate()) {
        status = 'Game over, ' + moveColor + ' is in checkmate.';
      } else if (game.in_draw()) {
        status = 'Game over, drawn position.';
      } else {
        status = moveColor + ' to move';
        if (game.in_check()) {
          status += ', ' + moveColor + ' is in check';
        }
      }

      document.getElementById('status').innerText = status;
    }

    var config = {
      draggable: true,
      position: 'start',
      onDragStart: onDragStart,
      onDrop: onDrop,
      onSnapEnd: onSnapEnd
    };
    $(document).ready(function() {
      board = Chessboard('board', config);
      updateStatus();

      // Control Buttons
      $('#flipBtn').on('click', function() { board.flip(); });
      $('#resetBtn').on('click', function() { game.reset(); board.start(); updateStatus(); });
    });
  </script>
</body>
</html>
