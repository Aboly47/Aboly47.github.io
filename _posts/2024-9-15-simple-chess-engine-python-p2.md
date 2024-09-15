# Writing a simple chess engine in Python - Part 2

In the [last post](https://aboly47.github.io/2024/09/15/simple-chess-engine-python-p1.html), we set up our workspace. In this part, we want to start to write some code.

## What do we need to write?

So, what do we need to write to have software that plays chess?

A chess engine is divided into a few parts:

* A board representation function. this keeps track of the board, the moves, the rules, and everything else related to the chess board. We will be using `python-chess` for this, but I might write one from scratch later. (If you're interested in writing one, search for chess board representation using bitboards.)
* A search function. This part maps out all of the moves possible from this position until a certain **depth**.
* An evaluation function. This part looks at a position and tells you how good is it for each player. for example, stockfish saying a position is **+2.5** means it's better for white. this is in pawn units, which we'll discuss later.
* A time management algorithm, that tells us how much time we should spend on this move, depending on how complex and critical the position is.
* A UCI interface, which will let our engine communicate with chess GUIs like Pychess or cutechess.

let's implement the Evaluation function first.

## Evaluation - Material

The simplest evaluation function is to count the material on the board.

***Note:*** From now on, p: pawn, n: knight, b: bishop, r: rook, q: queen, k: king.

We know that in a chess game, some pieces are worth more than others. for example, a queen is way stronger than a knight. but, we can't just tell a computer a queen is worth **More**. We can, but it won't be practical when we're searching for the best move. So, we need to assign a value to every piece.

a simple approach would be this:

`p: 1, n: 3, b: 3, r: 5, q: 9, k: 0`

in this system, we set Pawn to one. this is a pawn unit. King is set to zero because we can't capture the king.

Now, while this system is okay for human evaluations, it's not so good for computers. why? because a knight is worth slightly less than a bishop, cause a bishop sees lots of squares. this is true for open positions, in closed positions, a knight is worth more because it can jump around and create complications. but, for the sake of simplicity, this is true for all positions, for now.

so, a better approach would be something like this:

`p: 100, n: 280, b: 320, r: 479, q: 929, k: 60000`

it kinda looks like we multiplied everything by 100 and then played around with the values (except for the king value), and that's exactly what happened.

this is called the ***centipawn unit***.

A pawn is worth 100 centipawns, A knight is worth 280 centipawns, and so on.

These values have been found by people and computers experimenting and tuning their values.

But why is the king worth 60000? Well, that's because we're writing a *king capture* engine.

Think about it, putting a king in check is like attacking any other piece. but the difference is, that you can let a piece die, but you can't let your king die, so you move your king. but, what if every move was also check and you didn't have a *legal* move? (a legal move is a move that doesn't put your own king in check.) well, then you'd have to let your king die. you could play any move, and your king would be captured on the next turn. so, our ultimate goal is to capture the king. so, the king must have a value, right? but why such a big number? the number needs to be bigger than *8p + 2n + 2b + 2r + 1q*, hence the 60000.

## Let's write some code, shall we?

I will be using python-chess for this.

```python
import chess

def evaluate_material(board):
    # Define the material values for each piece
    piece_values = {
        chess.PAWN: 100,
        chess.KNIGHT: 280,
        chess.BISHOP: 320,
        chess.ROOK: 479,
        chess.QUEEN: 929,
        chess.KING: 60000
    }

    # Initialize scores
    white_score = 0
    black_score = 0

    # Iterate through all pieces on the board
    for piece_type in piece_values.keys():
        white_count = len(board.pieces(piece_type, chess.WHITE))
        black_count = len(board.pieces(piece_type, chess.BLACK))

        # Update scores based on counts and piece values
        white_score += white_count * piece_values[piece_type]
        black_score += black_count * piece_values[piece_type]

    # Return the material score difference
    return white_score - black_score

# Example usage
if __name__ == "__main__":
    # Create a chessboard
    board = chess.Board()

    # Evaluate the material balance
    score = evaluate_material(board)
    print("Material score:", score)
```

***Note:*** I will be assuming you know how to write Python code. so, I will assume you know what `def something()` does, or what `print()` does, so on. if you don't know what something does, search on Google or write a comment below and I'll answer.

this code is pretty self explanatory. we define a dictionary containing the value for each piece, then get the piece on the board for white and black, multiply them by the values, sum them up for each color, and subtract them. this will give you a number for each position.

in this line:

```python
board = chess.Board()
```

`Board()` takes an argument `fen=`. so, you can pass the fen of any position in there to *evaluate* it. by default, it has the starting position fen, which is `rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1`.

for example, `rnbqkbnr/ppp1pppp/8/3P4/8/8/PPPP1PPP/RNBQKBNR b KQkq - 0 2` is a position where white is a pawn up. it's the Scandinavian defense. we can evaluate it by changing:

```python
board = chess.Board()
```

to:

```python
board = chess.Board(fen="rnbqkbnr/ppp1pppp/8/3P4/8/8/PPPP1PPP/RNBQKBNR b KQkq - 0 2")
```

run the code again and you should see:

```plaintext
Material score: 100
```

which is exactly what we want. :)

In this post, we built a simple evaluation function. in the next post, we'll build a simple alpha-beta search and find the ***best move*** using it.
