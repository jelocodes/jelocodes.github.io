---
layout: post
title:  "Ruby Game Theory: Tic-Tac-Toe with A.I."
date:   2017-06-24 21:52:28 -0400
---

![](https://i.ytimg.com/vi/6DGNZnfKYnU/hqdefault.jpg)
> Computer from the film 'War Games,' 1983

I recently built an unbeatable game of Tic-Tac-Toe (represented by the robot Bender from Futurama to remind us of the impending machine uprising). Tic-Tac-Toe is one of my favourite games, though not to play. As a playable game it verges on boringly predictable, but as an illustrative model for games and decision making, it's elegant because it's really simple to model. In fact, the rule set is so basic and the game board so small that when one has learned the patterns, one can effectively never lose.


![](http://i.imgur.com/QhgB5eJ.png?1)


Computer science and games have had a long and storied history. For awhile, an assumption that was popularly held was that to play and win at games, one required 'intelligence' and 'consciousness,' both deemed solely human traits. However, from Deep Blue dethroning Chess champion Garry Kasparov in '97, to IBM's Watson beating trivia expert Ken Jennings at Jeopardy in 2010, to just last March, when Google's DeepMind A.I. defeated world champion Go player Lee Sedol, advances in computing have shown that this isn't the case. 

To explain this, what is needed is a less vague explanation for how computers are able to 'win' at games other than subjective words like intelligence and consciousness. What is considerably less vague is the process of decision making (or functional intelligence), which can be modelled, and given enough information about the situation, programmed.

Game theory is the study of mathematical models that deals with rational decision making. In zero-sum games (where one either only wins or loses), everything we do (or choose not to do) can be defined as behavioural outputs based on reactions to environmental inputs (in this case, the state of the game) to gain benefit or avoid cost.

![](http://www.informationr.net/ir/9-4/p191fig3.gif)
> A simple model of intelligent decision making 

Games are environmental models where there are finite rulesets and possible outcomes. Therefore, we as programmers can encapsulate such rulesets and outcomes in our code for the computer to base its decisions on.

The following is my attempt at implementing my own somewhat convuluted (but successful) never losing Tic-Tac-Toe A.I. (because, you know, Chess and Go have been done). How I did it:

There is an algorithm called Minimax that is used for minimizing the possible loss for a worst case scenario. It can also deal with making decisions based on procuring the maximum gains possible (in which case it is called Maximin). It's pretty elegant, and it's formal definition is as follows:

![](http://i.imgur.com/KbXF1Qv.png)
> The Minimax algorithm, via Wikipedia

It took awhile to really understand it, but essentially, the algorithm involves keeping track of all potential moves that can be made based on the current state of the game (or, stated formally, all cumulative actions made by the opponent up to this point: a₋ᵢ), and then making the move that would be least costly compared to all the rest (aᵢ). It implements a score counter system, which assigns a numerical point for each possible move (either positive for moves approaching a win or negative for moves approaching a loss) in order for the computer to calculate the value of the least costly move to make that turn (vᵢ). 

I built the logic of my Tic-Tac-Toe game before deciding to add an A.I. player, and my implementation did not have points allocated to moves (I was too lazy to refactor). However, as there are only two real possible scoring outcomes in such a simple game: An ‘absolute win’ or ‘absolute loss,' and the game is over *immediately* after either of the two is achieved (instead of a continuum of worst, worse, better or best states with different point achievements), I figured Minimax point tracking *per move* wasn’t absolutely needed for my fast and dirty approach to the problem. Instead, just a general awareness of those two possible win/loss states are required, or rather, an awareness of all potential scenarios *just before* one of those states is achieved is required, which informs the A.I. whether to go in for the win or to block the opponent's win when the situation presents itself. That knowledge is all the A.I. really needs to win, or, at the very least, not lose. It's moves the rest of the time are simple, as the middle and corner spots on the board are known to be strategically best (The only time where this won't work is when there are two potential near win situations on the board, but if the A.I. is trained to always block a single near win scenario the moment it occurs, then two near win scenarios will never happen).

![](http://i.imgur.com/OhRUULG.png)
> Knowledge of all potential possible moves isn't necessary, just the immediate knowledge of when a win is imminent, and where to go in order to win or block.

A Tick-Tac-Toe board is a simple 3x3 board (9 squares or cells). My implementation of the game uses a Board class to display and keep track of all of the cells on the board. The state of each cell is represented as an array from index 0 to 8 (9 cells) mapped to the 9 cells on a game board, left to right, from index 0 being the very top left of the board, to index 8 being the very bottom right of the board. The #cells method shows the board's cell array, and the #display method shows the game board in it's current state. The board is initialized with 9 blank cells. 

Thus,

```
b = Board.new

b.cells
=> [" ", " ", " ", " ", " ", " ", " ", " "]

b.display
=>     |   |  
    -----------
       |   |  
    -----------
       |   |  
```

A method was also written for the Board class to check the current state of each of it's cells (whether it's "X," "O" or blank), called #position, as well as a method to check whether or not a cell on the board was blank was valid to move into called #valid_move?

An instance of a game is initialized via a Game class, and the behaviours of the players are maintained by instances of Player classes, which keeps track of the current game they belong to via a @game instance variable. Two subclasses inherit from the Player class: Human and Computer, with the Computer holding the logic to behave autonomously based on an algorithm so that it never loses. 

Now, remember our A.I. needs to constantly check the state of the game board as to whether or not there is *almost* a winning scenario so it can either win or block. It does not need to check *all* of the possible cell combinations on the board, just the ones where win combinations can ever happen. 

The Game class therefore contains a constant called WIN_COMBINATIONS that keeps track of, in each of its indexes, the board's cells that need to be filled by a row of like-tokens (all X's or all O's) in order to achieve a win for each game (it contains all possible winning combinations in either horizontal, vertical, or diagonal configurations).

```
WIN_COMBINATIONS =[
     [0, 1, 2], #horizontal win
     [3, 4, 5], #horizontal win
     [6, 7, 8], #horizontal win
     [0, 4, 8], #diagonal win
     [2, 4, 6], #diagonal win
     [0, 3, 6], #vertical win
     [1, 4, 7], #vertical win
     [2, 5, 8]  #vertical win
 ]
```

(Note: because this is a private constant, in order to expose this array to other instances (like the Player instances), a class getter method is needed within the Game class:

```
def self.win_combinations
    const_get("WIN_COMBINATIONS")
end
```

This will come in handy later.)

To illustrate this, in a situation where there are all "X"s in the three indexes stored in WIN_COMBINATION's first index, [0, 1, 2], the board looks like this:

```
     X | X | X  
    -----------
       |   |  
    -----------
       |   |  
```

A method called #win_indexes checks what the current state is ("X," "O" or blank) of each of the board's cells corresponding to the numbers in the WIN_COMBINATIONS array by iterating through the array and using the aforementioned #position method of the Board class to find that cell's current state in the board (whether it contains an "X," "O" or blank):

```
def win_indexes
    @win_indexes = WIN_COMBINATIONS.collect{|combo| combo.collect{|index| @board.position(index+1)}}
end
```

The return value of this method is an array of the current board's state, each index of which directly corresponds to the same index in the WIN_COMBINATIONS array. This represents whether or not a win is imminent for that row, column or diagonal. For instance, the X's, O's or blanks in the following array that the #win_indexes method returns,

```
  ==> [ ["X", " ", "X"],
        [" ", " " , "O"],
        ["O", "X", " "],
        ["O", " ", " "],
        ["X", " ', " "],
        [" ", " ", "O"],
        [" ", " ", "X"],
        ["X", "O", " "] ]
```

corresponds directly to the numerical positions in the WIN_COMBINATIONS array, index for index:

```
WIN_COMBINATIONS =[
     [0, 1, 2],
     [3, 4, 5],
     [6, 7, 8],
     [0, 4, 8],
     [2, 4, 6],
     [0, 3, 6],
     [1, 4, 7],
     [2, 5, 8]
 ]
 ```
 
The first "X" represents 0 on the board's cells array, the blank after it represents 1 on the board's cells array, the "X" after it represents 2, etc.

To reiterate, both of the two arrays above represent the same board, except one has the actual tokens on the board ("X", "O" or blank), while the other has their representative indexes corresponding to the board's cells array (0-8). Because of this one to one relationship, other methods can refer to each of the two arrays and find out information about the board.
 
In the particular above case, a win is imminent in the top row of the board, where an 'X' is needed in the top-middle (or index 1 of the board's cells array) to win.

For the A.I. to determine whether or not it's almost won, it needs to be aware of it's identity, as well as that of it's opponent's. An instance variable, called @token, representing either an "X" or "O", as well as a getter method to reveal this data, is necessary. Once that's in place, the A.I. can simply check each turn whether or not it, or it's opponent, has almost won, via a method called #almost_won. The method iterates through the #win_indexes array (which has the board's state, X's or O's, for each of the cells in the potential win combinations) and detects whether one of those indexes matches an *almost* winning condition (e.g. X X -, - X X, or X - X). 

```
def almost_won
   @game.win_indexes.index{|index| index == ["#{self.token}","#{self.token}"," "] || 
   index == ["#{self.token}"," ","#{self.token}"] || index == [" ","#{self.token}","#{self.token}"]} 
end
```

The #almost_won method's return value is the index number in the #win_index method's returned array that matches an *almost won* condition. If there is no *almost won* condition, the method returns 'nil.' 

In the example above, because ["X", " ", "X"] in the #win_index method's returned array matches an almost won condition, the #almost_won method returns it's index, 0. Through a similar method, the A.I. can also detect whether or not its opponent has almost won. 

Once it knows whether or not there is an *almost* winning condition for either itself or its opponent, it simply needs to make or block the winning move in the location that is required to win, which we can refer to as the 'winning cell'. We can obtain the location of this 'winning cell' by first getting the index of whatever cell remains blank (" ") in the *almost winning* combination.  

```
def win_or_block   
    winning_cell = @game.win_indexes[almost_won].index(" ")
		...
```

In the example above, the winning_cell would be 1, as the blank spot (" ") needed to win is in the 1st index of ["X", " ", "X"] ([0th index, 1st index, 2nd index]).

However, we still need to match that index with its corresponding location on the board. Recall that the indexes in the array returned by the #win_indexes method represent the same places on the board that the indexes in the WIN_COMBINATIONS array represent. Therefore, the tokens in index '0' of the #win_indexes array ["X", " ", "X"] are representatively equal to the numbers in index '0' of the WIN_COMBINATIONS array [0, 1, 2], which are representative of positions on the board. By first using the #almost_won returned index to match the correct WIN_COMBINATION array index (which contain the board's cell index numbers), then using the #winning_cell method's return value to find where the blank space is on the game board, we can locate where the A.I. needs to move:

```
    ...
    Game.win_combinations[almost_won][winning_cell]
end		
```

The A.I. can then use this returned integer value to make its move. 

If there is no *almost* winning move, the program needs to make the next best possible move: the middle, followed by any of the corners, then any remaining available blank cells on the board. Besides an immediate winning or losing condition, which is the first thing the A.I. should check for to block or win as soon as the opportunity presents itself, the fallback optimal moves can be represented in an array called 'optimal_moves,' which the A.I. can then iterate through to decide the next best move to make. 

```
optimal_moves = [4, 0, 2, 6, 8, @game.board.cells.index(" ")]
```

This way, the A.I. is always scanning the board for either a winning or losing condition to achieve or prevent a win, and making the best strategic moves otherwise.

After some refactoring, here's the final A.I. #move method for my implementation of an unbeatable Tic-Tac-Toe bot. It takes the current board as an argument, and then implements the methods above to make its move:

```
def move(board)
    if almost_won != nil
        win_or_block
    else
        optimal_moves.detect{|move| @game.board.valid_move?(move)}
end
```

Nothing magic, only logic. Perhaps one day, I'll refactor the A.I. some more to implement the Minimax algorithm, but until then, it remains the reigning champ. 
