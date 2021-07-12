# Ball Game Solver

This is a solver for the relatively popular "ball stacking" genre. Is has been specifically made with [this](https://play.google.com/store/apps/details?id=com.spicags.ballsort) android app in mind.

<img alt="Example level in the game" src="https://raw.githubusercontent.com/Matthias-Sleurink/Ball-Game-Solver-Java/master/example_image.png" width="200">

## How the game works
As visible above the game has a number of pots with balls in them. The goal is to have every pot sorted. You can do this by moving balls from one pot to either: an emtpy pot, or a pot with space, where the colour of the top ball is the same as the ball you are moving.


## How to use
### Input
When you start the solver you are guided through entering some information.

The first step is to enter the maximum amount of steps that the solver should try to find a solution.

The second step is to enter the ball and pot information. For every pot, split by newlines, you enter, from the bottom up, the number that every ball has split by spaces. To make this easy there is a setting in the game to make the balls not only show a colour but also a number.

For the above shown grid with max number of moves of 50:
```
> What should the max number of moves tried be? 50
Set max num to 50
> Enter numbers with spaces in between (left is bottom): 
1 2 3 4
5 3 2 2
4 3 1 5
4 3 4 5
1 5 2 1

Found 5 nonempty ballbuckets.
```
The 5 nonempty ballbuckets tells us that there are 5 buckets that have balls in them. When we inserted an empty line the solver added two extra emtpy ballbuckets and started the solve.

Input parsing has been written so that you can copy and paste the ball buckets into a text file and later on copy and paste it back into the application prompt. This simplifies finding the "best" solution.

### Output
For the given input above we see this output:
```
The solution takes 22 steps.
We attempted 26 tries.
Remember, solutions are played top down.
(0->5)
(0->6)
(1->0)
(1->0)
(1->6)
(2->1)
(3->1)
(3->5)
(3->6)
(3->5)
(0->3)
(0->3)
(0->3)
(0->2)
(4->0)
(2->0)
(2->0)
(2->6)
(2->5)
(4->3)
(4->1)
(4->0)
```

As the ouput said this solution is read from the top down. First we move from bucket 0 to 5, then from 0 to 6 and so on.

This part of the output is some performance information.
```
The solution takes 22 steps.
We attempted 26 tries.
```
It means that we tried 26 different moves, and that out solution is 22 moves long. We can thus conclude that there were only (26-22=)4 "backtracks" needed to find the right solution.

A user could now try shorter and shorter solutions to find the "best" solution, or enter this solution in the app.

For the example given above one will see the following results:

| Max moves 	|     Tries 	| Solution Length 	|
|-----------	|----------:	|-----------------	|
| 50        	|        26 	| 22              	|
| 22        	|        25 	| 22              	|
| 21        	|       200 	| 20              	|
| 20        	|       108 	| 20              	|
| 19        	|  78948425 	| 18              	|
| 18        	|  31692542 	| 18              	|
| 17        	| 155877821 	| No Solution     	|
| 16        	|  61915968 	| No Solution     	|
| 15        	|  24581406 	| No Solution     	|
| 14        	|   9718272 	| No Solution     	|
| 13        	|   3811205 	| No Solution     	|
| 12        	|   1476190 	| No Solution     	|
| 11        	|    566609 	| No Solution     	|
| 10        	|    216374 	| No Solution     	|
| 9         	|     81600 	| No Solution     	|
| 8         	|     30619 	| No Solution     	|
| 7         	|     11468 	| No Solution     	|
| 6         	|      4239 	| No Solution     	|
| 5         	|      1560 	| No Solution     	|
| 4         	|       580 	| No Solution     	|
| 3         	|       208 	| No Solution     	|
| 2         	|        62 	| No Solution     	|
| 1         	|        10 	| No Solution     	|
| 0         	|         0 	| No Solution     	|

As you can see the amount of tries goes up very quickly when a shorter solution is requested as long as one is possible.

The largest amount of steps can be seen when we set the maximum amount of moves just one below the minimum amount of moves that is required to finish the puzzle.
In this case we see that the solver has to try almost every possible combination of moves with said length. Which quickly explodes to being very large.

# How the Solver Works

This solver, written in Java, uses a Depth First search and uses Backtracking to find a solution. The main loop can be simplified to the following:

```java
class solver {
    boolean solve(Puzzle puzzle) {
        if (puzzle.solved()) return true;
        
        for (Move move : puzzle.movesPossible()) {
        	if (move.redudant()) continue;
        	
        	puzzle.doMove(move);
        	if (solve(puzzle)) {
        		return true;
	        }
        	puzzle.undoMove(move);
        }
    }
}
```
## Redundant (and looping) moves
We have a redundancy check. This prevents infinite loops and shortens certain solves.

The redundancy detection detects if any move is the direct opposite of a previous move up to three moves back, with checking to make sure that the moves in between don't remove this relationship.

This would mean that a moveset like 1>2, 2>3 is not accepted as 1>3 would be a faster alternative.
It also solves the issue of loops. A moveset like 1>2, 2>1 is also not accepted, as 1>1( = no move), would be a faster alternative.

This detection is created to work up to three moves back. This limit is because the further back you go the more calculations need to be done, and the less possible moves are removed from the future-moves list.