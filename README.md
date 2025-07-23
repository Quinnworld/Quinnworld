<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Simple Chess Page</title>
  <!-- Chessboard.js CSS -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.css">
  <style>
    body { display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; font-family: sans-serif; }
    #board { width: 400px; }
    #status { margin-top: 10px; text-align: center; }
  </style>
</head>
<body>
  <div>
    <div id="board"></div>
    <div id="status">Game in progress</div>
  </div>

  <!-- Dependencies -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.js"></script>

  <script>
    const game = new Chess();
    const board = Chessboard('board', {
      draggable: true,
      position: 'start',
      onDragStart: onDragStart,
      onDrop: onDrop,
      onSnapEnd: onSnapEnd
    });

    function onDragStart(source, piece, position, orientation) {
      // Do not pick up pieces if game is over
      if (game.game_over()) return false;
      // Only pick up pieces for the side to move
      if ((game.turn() === 'w' && piece.search(/^b/) !== -1) ||
          (game.turn() === 'b' && piece.search(/^w/) !== -1)) {
        return false;
      }
    }

    function onDrop(source, target) {
      // See if the move is legal
      const move = game.move({ from: source, to: target, promotion: 'q' });

      // Illegal move
      if (move === null) return 'snapback';

      updateStatus();
    }

    function onSnapEnd() {
      board.position(game.fen());
    }

    function updateStatus() {
      let status = '';
      if (game.in_checkmate()) {
        status = 'Game over, ' + (game.turn() === 'w' ? 'Black' : 'White') + ' wins by checkmate.';
      } else if (game.in_draw()) {
        status = 'Game over, drawn position.';
      } else {
        status = (game.turn() === 'w' ? 'White' : 'Black') + ' to move';
        if (game.in_check()) {
          status += ', ' + (game.turn() === 'w' ? 'White' : 'Black') + ' is in check';
        }
      }
      document.getElementById('status').textContent = status;
    }
  </script>
</body>
</html>
