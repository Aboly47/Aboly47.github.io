# Writing a simple chess engine in Python - Part 3

In the [last post](https://aboly47.github.io/2024/09/15/simple-chess-engine-python-p2.html), we wrote a function that evaluates the chess board and gives us a number. now, we want to write a search function that searches the tree of moves, evaluates every possible move, and gives us the best move.

## code cleanup

Last time we wrote this code:

```python
import chess

def evaluate(board: chess.Board) -> int:
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

we need to remove something from this. last time we were testing things and we needed an example of usage. we don't need it anymore. so, delete everything after (and including) `# Example usage`. so, your code should be:

```python
import chess

def evaluate(board: chess.Board) -> int:
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
```

save this in a file called `evaluation.py`. Now, let's get searching! (Alright, I'll stop with the puns.)

## Search

The most popular search algorithm for chess is alpha-beta pruning and that's what we'll be implementing.

![Alpha-Beta search Diagram](https://upload.wikimedia.org/wikipedia/commons/thumb/9/91/AB_pruning.svg/400px-AB_pruning.svg.png)

![Another diagram](https://upload.wikimedia.org/wikipedia/en/thumb/7/79/Minmaxab.gif/400px-Minmaxab.gif)

According to [Wikipedia](https://en.wikipedia.org/wiki/Alpha%E2%80%93beta_pruning):

Alpha-beta pruning is a search algorithm that seeks to decrease the number of nodes that are evaluated by the minimax algorithm in its search tree. It is an adversarial search algorithm used commonly for machine playing of two-player combinatorial games (Tic-tac-toe, Chess, Connect 4, etc.). It stops evaluating a move when at least one possibility has been found that proves the move to be worse than a previously examined move. Such moves need not be evaluated further. When applied to a standard minimax tree, it returns the same move as minimax would, but prunes away branches that cannot possibly influence the final decision.

The fuuudge?

anyway, after hours of debugging and re-writing and crying, I present to you, an alpha-beta search!

```python
import chess
from evaluation import evaluate

def alpha_beta(board, depth, alpha, beta, maximizing_player):
    if depth == 0 or board.is_game_over():
        return evaluate(board)

    if maximizing_player:
        max_eval = float('-inf')
        for move in board.legal_moves:
            board.push(move)
            eval = alpha_beta(board, depth - 1, alpha, beta, False)
            board.pop()
            max_eval = max(max_eval, eval)
            alpha = max(alpha, eval)
            if beta <= alpha:
                break  # Beta cut-off
        return max_eval
    else:
        min_eval = float('inf')
        for move in board.legal_moves:
            board.push(move)
            eval = alpha_beta(board, depth - 1, alpha, beta, True)
            board.pop()
            min_eval = min(min_eval, eval)
            beta = min(beta, eval)
            if beta <= alpha:
                break  # Alpha cut-off
        return min_eval

def best_move(board, depth):
    best_eval = float('-inf')
    best_move = None
    alpha = float('-inf')
    beta = float('inf')

    for move in board.legal_moves:
        board.push(move)
        eval = alpha_beta(board, depth - 1, alpha, beta, False)
        board.pop()

        if eval > best_eval:
            best_eval = eval
            best_move = move
            alpha = max(alpha, eval)

    return best_move

if __name__ == "__main__":
    # Example usage
    board = chess.Board()
    depth = 3  # Set the depth for the search
    move = best_move(board, depth)
    print(f"Best move: {move}")
```

Save this in a file called `search.py`, run it, and you'll see...

`Best move: g1h3`

Great! It works! now, try increasing the depth to 5. it should say:

`Best move: g2g3`

Great! now, try increasing it to 7 and run it. it'll take a while but it should say:

`Best move: g1f3`

why did it take so much to calculate depth 7? The evaluation function is so simple, it should calculate everything in no time!

BECAUSE PYTHON IS ***SLOW***!

Anyway, let me explain **the code** now.

## Alpha-beta function

```python
def alpha_beta(board, depth, alpha, beta, maximizing_player):
```

### Parameters:

* board: The current state of the chess board.

* depth: The maximum depth to which the algorithm will explore the game tree.

* alpha: The best value that the maximizing player (typically the player to move) can guarantee at this level or above.

* beta: The best value that the minimizing player can guarantee at this level or above.

* maximizing_player: A boolean indicating whether the current turn is for the maximizing player.

## Base Case

```python
if depth == 0 or board.is_game_over():
    return evaluate(board)
```

If the maximum depth is reached or the game is over (checkmate or stalemate), it calls the evaluate function to get the value of the current board position.

## Maximizing Player

```python
if maximizing_player:
    max_eval = float('-inf')
    for move in board.legal_moves:
        board.push(move)
        eval = alpha_beta(board, depth - 1, alpha, beta, False)
        board.pop()
        max_eval = max(max_eval, eval)
        alpha = max(alpha, eval)
        if beta <= alpha:
            break  # Beta cut-off
    return max_eval
```

If it's the maximizing player's turn:

Initialize max_eval to negative infinity.

Loop through all legal moves.

For each move, push it to the board (make the move) and recursively call alpha_beta for the next depth with maximizing_player set to False.

After evaluating the move, pop it off the board (undo the move).

Update max_eval to be the maximum of the current evaluation and the best evaluation found so far.

Update alpha to be the maximum of its current value and the evaluation of the move.

If beta is less than or equal to alpha, break out of the loop (beta cut-off), meaning further exploration of this branch is unnecessary.

## Minimizing Player

```python
else:
    min_eval = float('inf')
    for move in board.legal_moves:
        board.push(move)
        eval = alpha_beta(board, depth - 1, alpha, beta, True)
        board.pop()
        min_eval = min(min_eval, eval)
        beta = min(beta, eval)
        if beta <= alpha:
            break  # Alpha cut-off
    return min_eval
```

If it's the minimizing player's turn:

Initialize min_eval to positive infinity.

The logic is similar to the maximizing player, but it aims to minimize the evaluation score.

Update min_eval and beta accordingly.

Use alpha cut-off similarly to prune branches.

## Best Move Function

```python
def best_move(board, depth):
    best_eval = float('-inf')
    best_move = None
    alpha = float('-inf')
    beta = float('inf')

    for move in board.legal_moves:
        board.push(move)
        eval = alpha_beta(board, depth - 1, alpha, beta, False)
        board.pop()

        if eval > best_eval:
            best_eval = eval
            best_move = move
            alpha = max(alpha, eval)

    return best_move
```

This function determines the best move for the current player:

It initializes best_eval, best_move, alpha, and beta.

It iterates through all legal moves, pushing each move onto the board and calling alpha_beta to get the evaluation.

If the evaluation of the current move is better than the best found so far, it updates best_eval, best_move, and alpha.

Finally, it returns the best move found.

The rest of the code is similar to the previous post, just an example for testing.

In this post, we wrote a search function using python that searches all of the moves and gives us the ***best move***. in the next post, we will look for ways to optimize it and make it faster and more efficient.
