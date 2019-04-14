# xiangqi.js

[![Build Status](https://travis-ci.org/lengyanyu258/xiangqi.js.svg?branch=master)](https://travis-ci.org/lengyanyu258/xiangqi.js)

xiangqi.js is a Javascript xiangqi library that is used for xiangqi move
generation/validation, piece placement/movement, and check/checkmate/stalemate
detection - basically everything but the AI.

xiangqi.js has been extensively tested in node.js and most modern browsers.

## Example Code
The code below plays a complete game of xiangqi ... randomly.

```js
var Xiangqi = require('./xiangqi').Xiangqi;
var xiangqi = new Xiangqi();

while (!xiangqi.game_over()) {
  var moves = xiangqi.moves();
  var move = moves[Math.floor(Math.random() * moves.length)];
  xiangqi.move(move);
}
console.log(xiangqi.pgn());
```

Need a user interface?  Try Chris Oakman's excellent
[xiangqiboard.js](http://chessboardjs.com) library.  See
[xiangqiboard.js - Random vs Random](http://chessboardjs.com/examples#5002) for
an example integration of xiangqi.js with xiangqiboard.js.

## API

### Constructor: Xiangqi([ fen ])
The Xiangqi() constructor takes an optional parameter which specifies the board configuration
in [Forsyth-Edwards Notation](http://en.wikipedia.org/wiki/Forsyth%E2%80%93Edwards_Notation).

```js
// board defaults to the starting position when called with no parameters
var xiangqi = new Xiangqi();

// pass in a FEN string to load a particular position
var xiangqi = new Xiangqi('r1k4r/p2nb1p1/2b4p/1p1n1p2/2PP4/3Q1NB1/1P3PPP/R5K1 b - c3 0 19');
```

### .ascii()
Returns a string containing an ASCII diagram of the current position.

```js
var xiangqi = new Xiangqi();

// make some moves
xiangqi.move('e4');
xiangqi.move('e5');
xiangqi.move('f4');

xiangqi.ascii();
// -> '   +------------------------+
//      8 | r  n  b  q  k  b  n  r |
//      7 | p  p  p  p  .  p  p  p |
//      6 | .  .  .  .  .  .  .  . |
//      5 | .  .  .  .  p  .  .  . |
//      4 | .  .  .  .  P  P  .  . |
//      3 | .  .  .  .  .  .  .  . |
//      2 | P  P  P  P  .  .  P  P |
//      1 | R  N  B  Q  K  B  N  R |
//        +------------------------+
//          a  b  c  d  e  f  g  h'
```


### .board()
Returns an 2D array representation of the current position.  Empty squares are
represented by `null`.

```js
var xiangqi = new Xiangqi();

xiangqi.board();
// -> [[{type: 'r', color: 'b'},
        {type: 'n', color: 'b'},
        {type: 'b', color: 'b'},
        {type: 'q', color: 'b'},
        {type: 'k', color: 'b'},
        {type: 'b', color: 'b'},
        {type: 'n', color: 'b'},
        {type: 'r', color: 'b'}],
        [...],
        [...],
        [...],
        [...],
        [...],
        [{type: 'r', color: 'w'},
         {type: 'n', color: 'w'},
         {type: 'b', color: 'w'},
         {type: 'q', color: 'w'},
         {type: 'k', color: 'w'},
         {type: 'b', color: 'w'},
         {type: 'n', color: 'w'},
         {type: 'r', color: 'w'}]]
```


### .clear()
Clears the board.

```js
xiangqi.clear();
xiangqi.fen();
// -> '8/8/8/8/8/8/8/8 w - - 0 1' <- empty board
```

### .fen()
Returns the FEN string for the current position.

```js
var xiangqi = new Xiangqi();

// make some moves
xiangqi.move('e4');
xiangqi.move('e5');
xiangqi.move('f4');

xiangqi.fen();
// -> 'rnbqkbnr/pppp1ppp/8/4p3/4PP2/8/PPPP2PP/RNBQKBNR b KQkq f3 0 2'
```

### .game_over()
Returns true if the game has ended via checkmate, stalemate, draw, threefold repetition, or insufficient material. Otherwise, returns false.

```js
var xiangqi = new Xiangqi();
xiangqi.game_over();
// -> false

xiangqi.load('4k3/4P3/4K3/8/8/8/8/8 b - - 0 78');
xiangqi.game_over();
// -> true (stalemate)

xiangqi.load('rnb1kbnr/pppp1ppp/8/4p3/5PPq/8/PPPPP2P/RNBQKBNR w KQkq - 1 3');
xiangqi.game_over();
// -> true (checkmate)
```

### .get(square)
Returns the piece on the square:

```js
xiangqi.clear();
xiangqi.put({ type: xiangqi.PAWN, color: xiangqi.BLACK }, 'a5') // put a black pawn on a5

xiangqi.get('a5');
// -> { type: 'p', color: 'b' },
xiangqi.get('a6');
// -> null
```

### .history([ options ])
Returns a list containing the moves of the current game.  Options is an optional
parameter which may contain a 'verbose' flag.  See .moves() for a description of the
verbose move fields.

```js
var xiangqi = new Xiangqi();
xiangqi.move('e4');
xiangqi.move('e5');
xiangqi.move('f4');
xiangqi.move('exf4');

xiangqi.history();
// -> ['e4', 'e5', 'f4', 'exf4']

xiangqi.history({ verbose: true });
// -> [{ color: 'w', from: 'e2', to: 'e4', flags: 'b', piece: 'p', san: 'e4' },
//     { color: 'b', from: 'e7', to: 'e5', flags: 'b', piece: 'p', san: 'e5' },
//     { color: 'w', from: 'f2', to: 'f4', flags: 'b', piece: 'p', san: 'f4' },
//     { color: 'b', from: 'e5', to: 'f4', flags: 'c', piece: 'p', captured: 'p', san: 'exf4' }]
```

### .in_check()
Returns true or false if the side to move is in check.

```js
var xiangqi = new Xiangqi('rnb1kbnr/pppp1ppp/8/4p3/5PPq/8/PPPPP2P/RNBQKBNR w KQkq - 1 3');
xiangqi.in_check();
// -> true
```

### .in_checkmate()
Returns true or false if the side to move has been checkmated.

```js
var xiangqi = new Xiangqi('rnb1kbnr/pppp1ppp/8/4p3/5PPq/8/PPPPP2P/RNBQKBNR w KQkq - 1 3');
xiangqi.in_checkmate();
// -> true
```

### .in_draw()
Returns true or false if the game is drawn (50-move rule or insufficient material).

```js
var xiangqi = new Xiangqi('4k3/4P3/4K3/8/8/8/8/8 b - - 0 78');
xiangqi.in_draw();
// -> true
```

### .in_stalemate()
Returns true or false if the side to move has been stalemated.

```js
var xiangqi = new Xiangqi('4k3/4P3/4K3/8/8/8/8/8 b - - 0 78');
xiangqi.in_stalemate();
// -> true
```

### .in_threefold_repetition()
Returns true or false if the current board position has occurred three or more
times.

```js
var xiangqi = new Xiangqi('rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1');
// -> true
// rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq occurs 1st time
xiangqi.in_threefold_repetition();
// -> false

xiangqi.move('Nf3'); xiangqi.move('Nf6'); xiangqi.move('Ng1'); xiangqi.move('Ng8');
// rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq occurs 2nd time
xiangqi.in_threefold_repetition();
// -> false

xiangqi.move('Nf3'); xiangqi.move('Nf6'); xiangqi.move('Ng1'); xiangqi.move('Ng8');
// rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq occurs 3rd time
xiangqi.in_threefold_repetition();
// -> true
```

### .header()
Allows header information to be added to PGN output. Any number of key/value
pairs can be passed to .header().

```js
xiangqi.header('White', 'Robert James Fischer');
xiangqi.header('Black', 'Mikhail Tal');

// or

xiangqi.header('White', 'Morphy', 'Black', 'Anderssen', 'Date', '1858-??-??');
```

Calling .header() without any arguments returns the header information as an object.

```js
xiangqi.header();
// -> { White: 'Morphy', Black: 'Anderssen', Date: '1858-??-??' }
```

### .insufficient_material()
Returns true if the game is drawn due to insufficient material (K vs. K,
K vs. KB, or K vs. KN); otherwise false.

```js
var xiangqi = new Xiangqi('k7/8/n7/8/8/8/8/7K b - - 0 1');
xiangqi.insufficient_material()
// -> true
```

### .load(fen)
The board is cleared, and the FEN string is loaded.  Returns true if the position was
successfully loaded, otherwise false.

```js
var xiangqi = new Xiangqi();
xiangqi.load('4r3/8/2p2PPk/1p6/pP2p1R1/P1B5/2P2K2/3r4 w - - 1 45');
// -> true

xiangqi.load('4r3/8/X12XPk/1p6/pP2p1R1/P1B5/2P2K2/3r4 w - - 1 45');
// -> false, bad piece X
```

### .load_pgn(pgn, [ options ])
Load the moves of a game stored in
[Portable Game Notation](http://en.wikipedia.org/wiki/Portable_Game_Notation).
`pgn` should be a string. Options is an optional `object` which may contain
a string `newline_char` and a boolean `sloppy`.

The `newline_char` is a string representation of a valid RegExp fragment and is
used to process the PGN. It defaults to `\r?\n`. Special characters
should not be pre-escaped, but any literal special characters should be escaped
as is normal for a RegExp.  Keep in mind that backslashes in JavaScript strings
must themselves be escaped (see `sloppy_pgn` example below). Avoid using
a `newline_char` that may occur elsewhere in a PGN, such as `.` or `x`, as this
will result in unexpected behavior.

The `sloppy` flag is a boolean that permits xiangqi.js to parse moves in
non-standard notations. See `.move` documentation for more information about
non-SAN notations.

The method will return `true` if the PGN was parsed successfully, otherwise `false`.

```js
var xiangqi = new Xiangqi();
pgn = ['[Event "Casual Game"]',
       '[Site "Berlin GER"]',
       '[Date "1852.??.??"]',
       '[EventDate "?"]',
       '[Round "?"]',
       '[Result "1-0"]',
       '[White "Adolf Anderssen"]',
       '[Black "Jean Dufresne"]',
       '[ECO "C52"]',
       '[WhiteElo "?"]',
       '[BlackElo "?"]',
       '[PlyCount "47"]',
       '',
       '1.e4 e5 2.Nf3 Nc6 3.Bc4 Bc5 4.b4 Bxb4 5.c3 Ba5 6.d4 exd4 7.O-O',
       'd3 8.Qb3 Qf6 9.e5 Qg6 10.Re1 Nge7 11.Ba3 b5 12.Qxb5 Rb8 13.Qa4',
       'Bb6 14.Nbd2 Bb7 15.Ne4 Qf5 16.Bxd3 Qh5 17.Nf6+ gxf6 18.exf6',
       'Rg8 19.Rad1 Qxf3 20.Rxe7+ Nxe7 21.Qxd7+ Kxd7 22.Bf5+ Ke8',
       '23.Bd7+ Kf8 24.Bxe7# 1-0'];

xiangqi.load_pgn(pgn.join('\n'));
// -> true

xiangqi.fen();
// -> 1r3kr1/pbpBBp1p/1b3P2/8/8/2P2q2/P4PPP/3R2K1 b - - 0 24

xiangqi.ascii();
// -> '  +------------------------+
//     8 | .  r  .  .  .  k  r  . |
//     7 | p  b  p  B  B  p  .  p |
//     6 | .  b  .  .  .  P  .  . |
//     5 | .  .  .  .  .  .  .  . |
//     4 | .  .  .  .  .  .  .  . |
//     3 | .  .  P  .  .  q  .  . |
//     2 | P  .  .  .  .  P  P  P |
//     1 | .  .  .  R  .  .  K  . |
//       +------------------------+
//         a  b  c  d  e  f  g  h'


// Parse non-standard move formats and unusual line separators
var sloppy_pgn = ['[Event "Wijk aan Zee (Netherlands)"]',
  '[Date "1971.01.26"]',
  '[Result "1-0"]',
  '[White "Tigran Vartanovich Petrosian"]',
  '[Black "Hans Ree"]',
  '[ECO "A29"]',
  '',
  '1. Pc2c4 Pe7e5', // non-standard
  '2. Nc3 Nf6',
  '3. Nf3 Nc6',
  '4. g2g3 Bb4',    // non-standard
  '5. Nd5 Nxd5',
  '6. c4xd5 e5-e4', // non-standard
  '7. dxc6 exf3',
  '8. Qb3 1-0'
].join('|');

var options = {
  newline_char: '\\|', // Literal '|' character escaped
  sloppy: true
};

xiangqi.load_pgn(sloppy_pgn);
// -> false

xiangqi.load_pgn(sloppy_pgn, options);
// -> true

xiangqi.fen();
// -> 'r1bqk2r/pppp1ppp/2P5/8/1b6/1Q3pP1/PP1PPP1P/R1B1KB1R b KQkq - 1 8'
```

### .move(move, [ options ])
Attempts to make a move on the board, returning a move object if the move was
legal, otherwise null.  The .move function can be called two ways, by passing
a string in Standard Algebraic Notation (SAN):

```js
var xiangqi = new Xiangqi();

xiangqi.move('e4')
// -> { color: 'w', from: 'e2', to: 'e4', flags: 'b', piece: 'p', san: 'e4' }

xiangqi.move('nf6') // SAN is case sensitive!!
// -> null

xiangqi.move('Nf6')
// -> { color: 'b', from: 'g8', to: 'f6', flags: 'n', piece: 'n', san: 'Nf6' }
```

Or by passing .move() a move object (only the 'to', 'from', and when necessary
'promotion', fields are needed):

```js
var xiangqi = new Xiangqi();

xiangqi.move({ from: 'g2', to: 'g3' });
// -> { color: 'w', from: 'g2', to: 'g3', flags: 'n', piece: 'p', san: 'g3' }
```

An optional sloppy flag can be used to parse a variety of non-standard move
notations:

```js

var xiangqi = new Xiangqi();

// various forms of Long Algebraic Notation
xiangqi.move('e2e4', {sloppy: true});
// -> { color: 'w', from: 'e2', to: 'e4', flags: 'b', piece: 'p', san: 'e4' }
xiangqi.move('e7-e5', {sloppy: true});
// -> { color: 'b', from: 'e7', to: 'e5', flags: 'b', piece: 'p', san: 'e5' }
xiangqi.move('Pf2f4', {sloppy: true});
// -> { color: 'w', from: 'f2', to: 'f4', flags: 'b', piece: 'p', san: 'f4' }
xiangqi.move('Pe5xf4', {sloppy: true});
// -> { color: 'b', from: 'e5', to: 'f4', flags: 'c', piece: 'p', captured: 'p', san: 'exf4' }


// correctly parses incorrectly disambiguated moves
xiangqi = new Xiangqi('r2qkbnr/ppp2ppp/2n5/1B2pQ2/4P3/8/PPP2PPP/RNB1K2R b KQkq - 3 7');

xiangqi.move('Nge7');  // Ne7 is unambiguous because the knight on c6 is pinned
// -> null

xiangqi.move('Nge7', {sloppy: true});
// -> { color: 'b', from: 'g8', to: 'e7', flags: 'n', piece: 'n', san: 'Ne7' }
```
### .moves([ options ])
Returns a list of legal moves from the current position.  The function takes an optional parameter which controls the single-square move generation and verbosity.

```js
var xiangqi = new Xiangqi();
xiangqi.moves();
// -> ['a3', 'a4', 'b3', 'b4', 'c3', 'c4', 'd3', 'd4', 'e3', 'e4',
//     'f3', 'f4', 'g3', 'g4', 'h3', 'h4', 'Na3', 'Nc3', 'Nf3', 'Nh3']

xiangqi.moves({square: 'e2'});
// -> ['e3', 'e4']

xiangqi.moves({square: 'e9'}); // invalid square
// -> []

xiangqi.moves({ verbose: true });
// -> [{ color: 'w', from: 'a2', to: 'a3',
//       flags: 'n', piece: 'p', san 'a3'
//       # a captured: key is included when the move is a capture
//       # a promotion: key is included when the move is a promotion
//     },
//     ...
//     ]
```

The _piece_, _captured_, and _promotion_ fields contain the lowercase
representation of the applicable piece.

The _flags_ field in verbose mode may contain one or more of the following values:

- 'n' - a non-capture
- 'b' - a pawn push of two squares
- 'e' - an en passant capture
- 'c' - a standard capture
- 'p' - a promotion
- 'k' - kingside castling
- 'q' - queenside castling

A flag of 'pc' would mean that a pawn captured a piece on the 8th rank and promoted.

### .pgn([ options ])
Returns the game in PGN format. Options is an optional parameter which may include
max width and/or a newline character settings.

```js
var xiangqi = new Xiangqi();
xiangqi.header('White', 'Plunky', 'Black', 'Plinkie');
xiangqi.move('e4');
xiangqi.move('e5');
xiangqi.move('Nc3');
xiangqi.move('Nc6');

xiangqi.pgn({ max_width: 5, newline_char: '<br />' });
// -> '[White "Plunky"]<br />[Black "Plinkie"]<br /><br />1. e4 e5<br />2. Nc3 Nc6'
```

### .put(piece, square)
Place a piece on the square where piece is an object with the form
{ type: ..., color: ... }.  Returns true if the piece was successfully placed,
otherwise, the board remains unchanged and false is returned.  `put()` will fail
when passed an invalid piece or square, or when two or more kings of the
same color are placed.

```js
xiangqi.clear();

xiangqi.put({ type: xiangqi.PAWN, color: xiangqi.BLACK }, 'a5') // put a black pawn on a5
// -> true
xiangqi.put({ type: 'k', color: 'w' }, 'h1') // shorthand
// -> true

xiangqi.fen();
// -> '8/8/8/p7/8/8/8/7K w - - 0 0'

xiangqi.put({ type: 'z', color: 'w' }, 'a1') // invalid piece
// -> false

xiangqi.clear();

xiangqi.put({ type: 'k', color: 'w' }, 'a1')
// -> true

xiangqi.put({ type: 'k', color: 'w' }, 'h1') // fail - two kings
// -> false

```

### .remove(square)
Remove and return the piece on _square_.

```js
xiangqi.clear();
xiangqi.put({ type: xiangqi.PAWN, color: xiangqi.BLACK }, 'a5') // put a black pawn on a5
xiangqi.put({ type: xiangqi.KING, color: xiangqi.WHITE }, 'h1') // put a white king on h1

xiangqi.remove('a5');
// -> { type: 'p', color: 'b' },
xiangqi.remove('h1');
// -> { type: 'k', color: 'w' },
xiangqi.remove('e1');
// -> null
```

### .reset()
Reset the board to the initial starting position.

### .turn()
Returns the current side to move.

```js
xiangqi.load('rnbqkbnr/pppppppp/8/8/4P3/8/PPPP1PPP/RNBQKBNR b KQkq e3 0 1')
xiangqi.turn()
// -> 'b'
```

### .undo()
Takeback the last half-move, returning a move object if successful, otherwise null.

```js
var xiangqi = new Xiangqi();

xiangqi.fen();
// -> 'rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1'
xiangqi.move('e4');
xiangqi.fen();
// -> 'rnbqkbnr/pppppppp/8/8/4P3/8/PPPP1PPP/RNBQKBNR b KQkq e3 0 1'

xiangqi.undo();
// -> { color: 'w', from: 'e2', to: 'e4', flags: 'b', piece: 'p', san: 'e4' }
xiangqi.fen();
// -> 'rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1'
xiangqi.undo();
// -> null
```

### .validate_fen(fen):
Returns a validation object specifying validity or the errors found within the
FEN string.

```js
xiangqi.validate_fen('2n1r3/p1k2pp1/B1p3b1/P7/5bP1/2N1B3/1P2KP2/2R5 b - - 4 25');
// -> { valid: true, error_number: 0, error: 'No errors.' }

xiangqi.validate_fen('4r3/8/X12XPk/1p6/pP2p1R1/P1B5/2P2K2/3r4 w - - 1 45');
// -> { valid: false, error_number: 9,
//     error: '1st field (piece positions) is invalid [invalid piece].' }
```

## MUSIC

Musical support provided by:

- [The Grateful Dead](https://www.youtube.com/watch?feature=player_detailpage&v=ANF6qanEB7s#t=2999)
- [Umphrey's McGee](http://www.youtube.com/watch?v=jh-1fFWkSdw)

## BUGS

- The en passant square and castling flags aren't adjusted when using the put/remove functions (workaround: use .load() instead)

## TODO

- Support PGN
- Investigate the use of piece lists (this may shave a few cycles off
  generate_moves() and attacked()).
- Refactor API to use camelCase - yuck.
- Add more robust FEN validation.
