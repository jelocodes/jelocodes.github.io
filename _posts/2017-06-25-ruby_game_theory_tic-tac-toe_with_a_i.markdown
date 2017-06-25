---
layout: post
title:  "Ruby Game Theory: Tic-Tac-Toe with A.I."
date:   2017-06-24 21:52:28 -0400
---

![](https://i.ytimg.com/vi/6DGNZnfKYnU/hqdefault.jpg)
> Computer from the film 'War Games,' 1983

As an activity in putting OO Ruby into practice, I recently built an unbeatable game of Tic-Tac-Toe (represented by the robot Bender from Futurama to remind us of the impending machine uprising). Tic-Tac-Toe is one of my favourite games, though not to play. As a playable game it verges on boringly predictable, but as an illustrative model for games and decision making, it's elegant because it's really simple to model. In fact, the rule set is so basic and the game board so small that when one has thoroughly learned the patterns, one can effectively never lose.


![](http://i.imgur.com/QhgB5eJ.png?1)


Computer science and games have had a long and storied history. For awhile, humans in general held the assumption that to play and win at games, one required 'intelligence' and 'consciousness,' deemed solely human traits. However, from Deep Blue dethroning Chess champion Kasparov in '97, to IBM's Watson beating trivia expert Ken Jennings at Jeopardy in 2010, to just last March, when Google's DeepMind A.I. defeated world champion Go player Lee Sedol, advances in computing have shown that this isn't the case. 

To explain this, what is needed is a less vague explanation for how computers are able to 'win' at games other than subjective words like intelligence and consciousness. What is considerably less vague is the process of decision making (or functional intelligence), which can be modelled, and given enough information about the situation, programmed.

Game theory is a branch of mathematics that deals with rational decision making. In zero-sum games (where one either only wins or loses), everything we do (or choose not to do) can be defined as behavioural outputs based on reactions to environmental inputs (in this case, the state of the game) to gain benefit or avoid cost.

![](http://www.informationr.net/ir/9-4/p191fig3.gif)
> A simple model of intelligent decision making by systems theorist E. Hartmann 

Games are environmental models where there are relatively small, finite rulesets and possible outcomes. Therefore, we as programmers can encapsulate such rulesets and outcomes in our code for the computer to base its decisions on.

The following is my first attempt at implementing my own somewhat convuluted but successful always winning strategy for a Tic-Tac-Toe A.I. (because, you know, Chess and Go have been done). How I did it:

There is an algorithm called [Minimax](http://en.wikipedia.org/wiki/Minimax) that is used for minimizing the possible loss for a worst case scenario. It can also deal with making decisions based on procuring the maximum gains possible, in which it is called Maximin. It's pretty elegant, and it's formal definition is as follows:

![](http://i.imgur.com/KbXF1Qv.png)
> From Wikipedia

It took awhile to really understand it, but essentially, the algorithm involves keeping track of all potential moves that can be made based on the current state of the game (or all cumulative actions made by the opponent up to this point: a₋ᵢ), and then making the move that would be least costly compared to all the rest (aᵢ). It implements a score counter system, which assigns a numerical point for each possible move (either positive for wins and negative for losses) in order for the computer to calculate the least costly move to make (vᵢ). 

My implementation of Tic-Tac-Toe did not have points allocated to moves (and I was too lazy to refactor), however, as there are only two real possible scoring outcomes in such a simple game: an 'absolute win' or 'absolute loss,' and the game is over *immediately* after either of the two is achieved (instead of a continuum of worst, worse, better or best states for point-achievement), I figured the Minimax point tracking wasn't *absolutely* needed for my first crack at the problem. Instead, just a general awareness of those two possible win/loss states are required, or rather, an awareness of the board *just before* one of those states is achieved is required, which informs the A.I. whether to go in for the win or to block the opponent's win when the situation arises. 

A Tick-Tac-Toe board is a simple 3x3 board (9 squares or cells). My implementation of the game uses a Board class to display and keep track of all of the cells on the board. The state of each cell is represented as an array from index 0 to 8 (9 squares) mapped to the 9 squares on a game board, left to right, from index 0 being the very top left of the board, to index 8 being the very bottom right of the board. The board is initialized with 9 blank squares. 

```
class Board
    attr_accessor :cells
		
    def initialize
		@cells = []
        9.times{@cells << " "}
    end
		
    def display
        puts " #{cells[0]} | #{cells[1]} | #{cells[2]} "
        puts "-----------"
        puts " #{cells[3]} | #{cells[4]} | #{cells[5]} "
        puts "-----------"
        puts " #{cells[6]} | #{cells[7]} | #{cells[8]} "
    end
		
end

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

A method was also written, which we won't describe here, for the game to be able to check the current state of each position on the board (whether it's "X," "O" or blank), called #position.

An instance of a game is initialized via a Game class, and the behaviours of the players are maintained by instances of Player classes. Two subclasses inherit from the Player class: Human and Computer, with the Computer holding the logic to behave autonomously based on an algorithm so that it never loses. 

Now, remember our A.I. needs to constantly check the state of the game board as to whether or not there is *almost* a winning scenario so it can either win or block. It does not need to check *all* of the possible cell combinations on the board, just the ones where win combinations can ever happen. 

The Game class therefore contains a constant called WIN_COMBINATIONS that keeps track of, in each of its indexes, the board's cells that need to be filled by a row of similar tokens ("X" or "O") in order to achieve a win.

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

Note: because this is a private constant, in order to expose this array to other instances (like the Player instances), I need a getter method within the Game class:

```
def self.win_combinations
    const_get("WIN_COMBINATIONS")
end
```

To illustrate this, all "X"s in WIN_COMBINATION's first index, [0,1, 2], would correspond to the indexes in the cell's array and represent the following on the board:

```
     X | X | X  
    -----------
       |   |  
    -----------
       |   |  
```

A method called #win_indexes checks what the current state ("X," "O" or blank) is of each of the board cells in WIN_COMBINATIONS by iterating through the WIN_COMBINATIONS array and, for each index, using the #position method of the Board class to find that cell's current state:

```
def win_indexes
    @win_indexes = WIN_COMBINATIONS.collect{|combo| combo.collect{|index| @board.position(index+1)}}
end
```

The return value of this method is an array of the current board's state, each index of which directly corresponds to the WIN_COMBINATIONS array, and represents whether or not a win is imminent. For instance:

```
  ==>   ["X", " ", "X"],
        [" ", " " , "O"],
        ["O", "X", " "],
        ["O", " ", " "],
        ["X", " ', " "],
        [" ", " ", "O"],
        [" ", " ", "X"],
        ["X", "O", " "]
```

corresponds directly to the positions in:

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
 
In this case, a win is imminent in the top row of the board, with a middle 'X' (in index 1 of the cells array) needed to win.

For the A.I. program to determine whether or not it's won, it needs to be aware of it's identity, as well as it's opponent's. An instance variable called @token representing whether or not it's an "X" or "O", as well as a getter method to reveal this data, is necessary. Once that's in place, it can simply check each turn whether or not it's almost won by iterating through the win_indexes array and *detecting* whether one of those indexes matches an *almost* winning condition for it's own token.

```
def almost_won
   @game.win_indexes.index{|index| index == ["#{self.token}","#{self.token}"," "] || 
   index == ["#{self.token}"," ","#{self.token}"] || index == [" ","#{self.token}","#{self.token}"]} 
end
```

Through similar means, the A.I. can also detect whether or not it's opponent has almost won. Once it detects whether or not there is an *almost* winning condition, it simply needs to make or block the winning move in the location (which we can refer to as the 'winning cell'). It can obtain this via getting the index of whatever cell remains blank (" ") in the almost winning array,

```
def win_or_block   
    winning_cell = [almost_won].index(" ")
```

and then matching it up with its corresponding board index via the Game class's WIN_COMBINATIONS constant. 

```
    Game.win_combinations[almost_won][winning_cell]
```

If there is no almost winning move, the program needs to make the next best possible move, the middle, followed by any of the corners, then any remaining available blank square on the board. Besides an immediate winning or losing condition, the optimal moves on the board can be represented in an array 'optimal_moves.' 

```
optimal_moves = [5, 1, 3, 7, @game.board.cells.index(" ")]
```

This way, the A.I. is always scanning the board for the binary either winning or losing conditions to achieve or prevent a win, and making the best strategic moves otherwise.

After some more refactoring and with the logic abstracted away, here's the final A.I. #move method for my first successful implementation of an unbeatable Tic-Tac-Toe 'bot. Perhaps I'll refactor the code some more one day to implement the more generalized, simple Minimax algorithm. 

```
def move(board)
    if almost_won != nil
        win_or_block
    else
        optimal_moves.detect{|move| @game.board.valid_move?(move)}
end
```



