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

Need a user interface?  Try [lengyanyu258](https://github.com/lengyanyu258)'s excellent
[xiangqiboard.js](https://lengyanyu258.github.io/xiangqiboardjs/index.html) library.  See
[xiangqiboard.js - Random vs Random](https://lengyanyu258.github.io/xiangqiboardjs/examples.html#5002) for
an example integration of xiangqi.js with xiangqiboard.js.

## API

### Constructor: Xiangqi([ fen ])
The Xiangqi() constructor takes an optional parameter which specifies the board configuration
in [Forsyth-Edwards Notation](http://en.wikipedia.org/wiki/Forsyth%E2%80%93Edwards_Notation).
But there are many differences between [Chess FEN](http://www.xqbase.com/protocol/pgnfen2.htm) and [Xiangqi FEN](http://www.xqbase.com/protocol/cchess_fen.htm).

```js
// board defaults to the starting position when called with no parameters
var xiangqi = new Xiangqi();

// pass in a FEN string to load a particular position
var xiangqi = new Xiangqi('5kC2/4a1N2/3a5/9/9/9/9/r3C4/4p4/2rK4R r - - 0 1');
```

### .ascii()
Returns a string containing an ASCII diagram of the current position.

```js
var xiangqi = new Xiangqi();

// make some moves
xiangqi.move('e3e4');
xiangqi.move('e6e5');
xiangqi.move('g3g4');

xiangqi.ascii();
// -> '   +---------------------------+
//      9 | r  n  b  a  k  a  b  n  r |
//      8 | .  .  .  .  .  .  .  .  . |
//      7 | .  c  .  .  .  .  .  c  . |
//      6 | p  .  p  .  .  .  p  .  p |
//      5 | .  .  .  .  p  .  .  .  . |
//      4 | .  .  .  .  P  .  P  .  . |
//      3 | P  .  P  .  .  .  .  .  P |
//      2 | .  C  .  .  .  .  .  C  . |
//      1 | .  .  .  .  .  .  .  .  . |
//      0 | R  N  B  A  K  A  B  N  R |
//        +---------------------------+
//          a  b  c  d  e  f  g  h  i
```


### .board()
Returns an 2D array representation of the current position.  Empty squares are
represented by `null`.

```js
var xiangqi = new Xiangqi();

xiangqi.board();
// -> [[{type: 'r', color: 'b'},
//      {type: 'n', color: 'b'},
//      {type: 'b', color: 'b'},
//      {type: 'a', color: 'b'},
//      {type: 'k', color: 'b'},
//      {type: 'a', color: 'b'},
//      {type: 'b', color: 'b'},
//      {type: 'n', color: 'b'},
//      {type: 'r', color: 'b'}],
//      [...],
//      [...],
//      [...],
//      [...],
//      [...],
//      [...],
//      [...],
//      [...],
//      [{type: 'r', color: 'r'},
//       {type: 'n', color: 'r'},
//       {type: 'b', color: 'r'},
//       {type: 'a', color: 'r'},
//       {type: 'k', color: 'r'},
//       {type: 'a', color: 'r'},
//       {type: 'b', color: 'r'},
//       {type: 'n', color: 'r'},
//       {type: 'r', color: 'r'}]]
```


### .clear()
Clears the board.

```js
xiangqi.clear();
xiangqi.fen();
// -> '9/9/9/9/9/9/9/9/9/9 r - - 0 1' <- empty board
```

### .fen()
Returns the FEN string for the current position.

```js
var xiangqi = new Xiangqi();

// make some moves
xiangqi.move('e3e4');
xiangqi.move('e6e5');
xiangqi.move('g3g4');

xiangqi.fen();
// -> 'rnbakabnr/9/1c5c1/p1p3p1p/4p4/4P1P2/P1P5P/1C5C1/9/RNBAKABNR b - - 3 2'
```

### .game_over()
Returns true if the game has ended via checkmate, stalemate, draw, threefold repetition, or insufficient material. Otherwise, returns false.

```js
var xiangqi = new Xiangqi();
xiangqi.game_over();
// -> false

xiangqi.load('3aca3/1Cnrk4/b3r4/2p1n4/2b6/9/9/9/4C4/ppppcK3 b - - 0 1');
xiangqi.game_over();
// -> true (stalemate)

xiangqi.load('rnbakab1r/9/1c5c1/p1p5p/4p1p2/4P1P2/P1P3nCP/1C3A3/4NK3/RNB2AB1R r - - 0 1');
xiangqi.game_over();
// -> true (checkmate)
```

### .get(square)
Returns the piece on the square:

```js
xiangqi.clear();
xiangqi.put({ type: xiangqi.PAWN, color: xiangqi.BLACK }, 'a5'); // put a black pawn on a5

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
xiangqi.move('e3e4');
xiangqi.move('e6e5');
xiangqi.move('g3g4');
xiangqi.move('e5e4');

xiangqi.history();
// -> [ 'e3e4', 'e6e5', 'g3g4', 'e5e4' ]

xiangqi.history({ verbose: true });
// -> [{ color: 'r', from: 'e3', to: 'e4', flags: 'n', piece: 'p', iccs: 'e3e4' },
//     { color: 'b', from: 'e6', to: 'e5', flags: 'n', piece: 'p', iccs: 'e6e5' },
//     { color: 'r', from: 'g3', to: 'g4', flags: 'n', piece: 'p', iccs: 'g3g4' },
//     { color: 'b', from: 'e5', to: 'e4', flags: 'c', piece: 'p', captured: 'p', iccs: 'e5e4' }]
```

### .in_check()
Returns true or false if the side to move is in check.

```js
var xiangqi = new Xiangqi('rnbakab1r/9/1c5c1/p1p5p/4p1p2/4P1P2/P1P3nCP/1C3A3/4NK3/RNB2AB1R r - - 0 1');
xiangqi.in_check();
// -> true
```

### .in_checkmate()
Returns true or false if the side to move has been checkmated.

```js
var xiangqi = new Xiangqi('rnbakab1r/9/1c5c1/p1p5p/4p1p2/4P1P2/P1P3nCP/1C3A3/4NK3/RNB2AB1R r - - 0 1');
xiangqi.in_checkmate();
// -> true
```

### .in_draw()
Returns true or false if the game is drawn (50-move rule or insufficient material).

```js
var xiangqi = new Xiangqi('4k1b2/9/4b4/9/9/9/9/4B4/9/4K1B2 r - - 0 1');
xiangqi.in_draw();
// -> true
```

### .in_stalemate()
Returns true or false if the side to move has been stalemated.

```js
var xiangqi = new Xiangqi('3aca3/1Cnrk4/b3r4/2p1n4/2b6/9/9/9/4C4/ppppcK3 b - - 0 1');
xiangqi.in_stalemate();
// -> true
```

### .in_threefold_repetition()
Returns true or false if the current board position has occurred three or more
times.

```js
var xiangqi = new Xiangqi('rnbakabnr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR r - - 0 1');
// -> true
// rnbakabnr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR r - - occurs 1st time
xiangqi.in_threefold_repetition();
// -> false

xiangqi.move('h0g2'); xiangqi.move('h9g7'); xiangqi.move('g2h0'); xiangqi.move('g7h9');
// rnbakabnr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR r - - occurs 2nd time
xiangqi.in_threefold_repetition();
// -> false

xiangqi.move('h0i2'); xiangqi.move('h9i7'); xiangqi.move('i2h0'); xiangqi.move('i7h9');
// rnbakabnr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR r - - occurs 3rd time
xiangqi.in_threefold_repetition();
// -> true
```

### .header()
Allows header information to be added to PGN output. Any number of key/value
pairs can be passed to .header().

```js
xiangqi.header('Red', '吕钦');
xiangqi.header('Black', '许银川');

// or

xiangqi.header('Red', '许银川', 'Black', '聂卫平', 'Date', '1999.12.09');
```

Calling .header() without any arguments returns the header information as an object.

```js
xiangqi.header();
// -> { Red: '许银川', Black: '聂卫平', Date: '1999.12.09' }
```

### .insufficient_material()
Returns true if the game is drawn due to insufficient material (K vs. K,
K vs. KB, or K vs. KN); otherwise false.

```js
var xiangqi = new Xiangqi('4k1b2/9/4b4/9/9/9/9/4B4/9/4K1B2 r - - 0 1');
xiangqi.insufficient_material()
// -> true
```

### .load(fen)
The board is cleared, and the FEN string is loaded.  Returns true if the position was
successfully loaded, otherwise false.

```js
var xiangqi = new Xiangqi();
xiangqi.load('1nbakabn1/9/1c5c1/p1p3p1p/4p4/4P4/P1P3P1P/1C5C1/9/1NBAKABN1 b - - 1 2');
// -> true

xiangqi.load('1nbakabn1/9/1c5c1/p1p3p1X/4p4/4P4/P1P3P1P/1C5C1/9/1NBAKABN1 b - - 1 2');
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

xiangqi.move('e3e4');
// -> { color: 'r', from: 'e3', to: 'e4', flags: 'n', piece: 'p', iccs: 'e3e4' }

xiangqi.move('nf6'); // SAN is case sensitive!!
// -> null

xiangqi.move('Nf6');
// -> { color: 'b', from: 'g8', to: 'f6', flags: 'n', piece: 'n', san: 'Nf6' }
```

Or by passing .move() a move object (only the 'to', 'from', and when necessary
'promotion', fields are needed):

```js
var xiangqi = new Xiangqi();

xiangqi.move({ from: 'g3', to: 'g4' });
// -> { color: 'r', from: 'g3', to: 'g4', flags: 'n', piece: 'p', iccs: 'g3g4' }
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
// -> [ 'a3a4', 'c3c4', 'e3e4', 'g3g4', 'i3i4', 'b2b3', 'b2b4', 'b2b5', 'b2b6', 'b2b9', 'b2b1',
//      'b2a2', 'b2c2', 'b2d2', 'b2e2', 'b2f2', 'b2g2', 'h2h3', 'h2h4', 'h2h5', 'h2h6', 'h2h9',
//      'h2h1', 'h2g2', 'h2f2', 'h2e2', 'h2d2', 'h2c2', 'h2i2', 'a0a1', 'a0a2', 'b0a2', 'b0c2',
//      'c0a2', 'c0e2', 'd0e1', 'e0e1', 'f0e1', 'g0e2', 'g0i2', 'h0g2', 'h0i2', 'i0i1', 'i0i2' ]

xiangqi.moves({square: 'b0'});
// -> [ 'b0a2', 'b0c2' ]

xiangqi.moves({square: 'e9'}); // invalid square
// -> []

xiangqi.moves({ verbose: true });
// -> [{ color: 'r', from: 'a3', to: 'a4',
//       flags: 'n', piece: 'p', iccs 'a3a4'
//       # a captured: key is included when the move is a capture
//     },
//     ...
//     ]

xiangqi.moves({ verbose: true, opponent: true });
// -> [{ color: 'b', from: 'a9', to: 'a8',
//       flags: 'n', piece: 'r', iccs 'a9a8'
//       # a captured: key is included when the move is a capture
//     },
//     ...
//     ]
```

The _piece_, _captured_, and _promotion_ fields contain the lowercase
representation of the applicable piece.

The _flags_ field in verbose mode may contain one or more of the following values:

- 'n' - a non-capture
- 'c' - a standard capture

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

xiangqi.put({ type: xiangqi.PAWN, color: xiangqi.BLACK }, 'a5'); // put a black pawn on a5
// -> true
xiangqi.put({ type: 'k', color: 'r' }, 'e1'); // shorthand
// -> true

xiangqi.fen();
// -> '9/9/9/9/p8/9/9/9/4K4/9 r - - 0 1'

xiangqi.put({ type: 'z', color: 'w' }, 'a1'); // invalid piece
// -> false

xiangqi.clear();

xiangqi.put({ type: 'k', color: 'r' }, 'e1');
// -> true

xiangqi.put({ type: 'k', color: 'r' }, 'f1') // fail - two kings
// -> false

```

### .remove(square)
Remove and return the piece on _square_.

```js
xiangqi.clear();
xiangqi.put({ type: xiangqi.PAWN, color: xiangqi.BLACK }, 'a5'); // put a black pawn on a5
xiangqi.put({ type: xiangqi.KING, color: xiangqi.RED }, 'e1'); // put a red king on e1

xiangqi.remove('a5');
// -> { type: 'p', color: 'b' },
xiangqi.remove('e1');
// -> { type: 'k', color: 'r' },
xiangqi.remove('h1');
// -> null
```

### .reset()
Reset the board to the initial starting position.

### .turn()
Returns the current side to move.

```js
xiangqi.load('rnbakabnr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR r - - 0 1');
xiangqi.turn();
// -> 'r'
```

### .undo()
Takeback the last half-move, returning a move object if successful, otherwise null.

```js
var xiangqi = new Xiangqi();

xiangqi.fen();
// -> 'rnbakabnr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR r - - 0 1'
xiangqi.move('e3e4');
xiangqi.fen();
// -> 'rnbakabnr/9/1c5c1/p1p1p1p1p/9/4P4/P1P3P1P/1C5C1/9/RNBAKABNR b - - 1 1'

xiangqi.undo();
// -> { color: 'r', from: 'e3', to: 'e4', flags: 'n', piece: 'p', iccs: 'e3e4' }
xiangqi.fen();
// -> 'rnbakabnr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR r - - 0 1'
xiangqi.undo();
// -> null
```

### .validate_fen(fen):
Returns a validation object specifying validity or the errors found within the
FEN string.

```js
xiangqi.validate_fen('rnbakabnr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR r - - 0 1');
// -> { valid: true, error_number: 0, error: 'No errors.' }

xiangqi.validate_fen('rnbaka?nr/9/1c5c1/p1p1p1p1p/9/9/P1P1P1P1P/1C5C1/9/RNBAKABNR r - - 0 1');
// -> { valid: false, error_number: 9,
//     error: '1st field (piece positions) is invalid [invalid piece].' }
```

## MUSIC

Musical support provided by:

- [The Grateful Dead](https://www.youtube.com/watch?feature=player_detailpage&v=ANF6qanEB7s#t=2999)
- [Umphrey's McGee](http://www.youtube.com/watch?v=jh-1fFWkSdw)

## BUGS

- ~~The en passant square and castling flags aren't adjusted when using the put/remove functions (workaround: use .load() instead)~~

## TODO

- Support PGN
- Investigate the use of piece lists (this may shave a few cycles off
  generate_moves() and attacked()).
- Refactor API to use camelCase - yuck.
- Add more robust FEN validation.
