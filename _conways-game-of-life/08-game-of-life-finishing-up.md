---
title: "The Finish Line"
permalink: /tdd/examples/conways-game-of-life/the-finish-line
last_modified_at: 2020-06-16
---
Continuing from our last session I want to take an opportunity to do a bit of refactoring now that our tests are all
passing. One thing that has been bothering me is the lack of a domain concept for either the *alive* or *dead* cell
states. The concept is implicitly there by using the *1* and *0* in our code, but I would like to make this idea more
explicit by introducing named constants to replace the numeric literals.

```java
class GameOfLife {
    private static final int ALIVE = 1;
    private static final int DEAD = 0;

    ....
}
```
Along these same lines, I would also like to extract our logic for deciding if a live cell should remain alive in the
next generation into a method. No, the boolean logic isn't complex, but it also doesn't explain itself in the language
of the problem either. Extracting the logic into a method allows us to name the condition by which a cell will continue
living.

```java
class GameOfLife {
    ....

    private int[][] nextGeneration(int[][] grid) {
        for(int x = 0; x < grid.length; x++) {
            for(int y = 0; y < grid[x].length; y++) {
                int aliveNeighbors = countAliveNeighbors(grid, x, y);
                if(isAlive(grid, x, y)) {
                    if (populationIsStable(aliveNeighbors)) {
                        nextGeneration[x][y] = ALIVE;
                    } else {
                        ...
                    }
                } else {
                    ...
                }
            }
        }
    }

    private boolean populationIsStable(int aliveNeighbors) {
        return aliveNeighbors == 2 || aliveNeighbors == 3;
    }
    ....
}
```

And we're going to do the same thing for the times when a dead cell is made alive in the next generation.

```java
class GameOfLife {
    ....

    private int[][] nextGeneration(int[][] grid) {
        for(int x = 0; x < grid.length; x++) {
            for(int y = 0; y < grid[x].length; y++) {
                int aliveNeighbors = countAliveNeighbors(grid, x, y);
                if(isAlive(grid, x, y)) {
                    ...
                } else {
                    if(populationIsExpanding(aliveNeighbors)) {
                        nextGeneration[x][y] = ALIVE;
                    }
                }
            }
        }
    }

    private boolean populationIsExpanding(int aliveNeighbors) {
        return aliveNeighbors == 3;
    }

    ....
}
```

I'm happy with where the code is at the moment, so let's shift our focus back to our tests. I think this is a good time 
to re-enable *'liveCellWithSingleLiveNeighborsDiesInNextGeneration'* that we had to disable a little while back.

```java
class GameOfLifeTest {
    ...    

    @Test
    void liveCellWithSingleLiveNeighborsDiesInNextGeneration() {
        var grid = new int[][] {
                {1,1,0},
                {0,0,0},
                {0,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var expectedNextGeneration = new int[][] {
                {0,0,0},
                {0,0,0},
                {0,0,0}
        };
        assertArrayEquals(expectedNextGeneration, nextGeneration);
    }
}
```

With the test re-enabled it now passes! This was a bit unexpected, honestly, as we don't have this use case explicitly 
implemented. It does make sense, though, as any unassigned cell values will default to 0, which is used to represent a 
dead cell.

Looking back to our list of scenarios I see the next on the list is *"live cell with three live neighbors, should live"*.
We've actually already handled this use case with an existing test, so we'll move on the next one which is
*"live cell with four live neighbors, should die"*.

```java
class GameOfLifeTest {
    ...

    @Test
    void liveCellWithFourLiveNeighborsDiesInNextGeneration() {
        var grid = new int[][] {
                {1,1,0},
                {0,1,0},
                {0,1,1}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var expectedNextGeneration = new int[][] {
                {1,1,0},
                {0,0,0},
                {0,1,1}
        };
        assertArrayEquals(expectedNextGeneration, nextGeneration);
    }
}
```

It also passes! This is a sign we may have a working solution. We should consider more of the dead cell scenarios before
we declare victory just yet. Looking at our list I see we still have *"// dead cell with one live neighbor, should 
remain dead"*, so let's write a test for it.

```java
class GameOfLifeTest {
    ...

    @Test
    void deadCellWithOneLiveNeighborsDiesInNextGeneration() {
        var grid = new int[][] {
                {0,1,0},
                {0,0,0},
                {0,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var expectedNextGeneration = new int[][] {
                {0,0,0},
                {0,0,0},
                {0,0,0}
        };
        assertArrayEquals(expectedNextGeneration, nextGeneration);
    }
}
```

This use case is also passing. I'm starting to feel pretty confident that we've arrive at a working solution to Conway's
Game of Life. However, before we leave I want to re-enable our final `@Disabled` test and verify that scenario is also
satisfied by our solution.

```java
class GameOfLifeTest {
    ...

    @Test
    void gridContainAllLiveCellsWillOnlyHaveCornersRemainingAlive() {
        var allAlive = new int[][]{
                {1,1,1},
                {1,1,1},
                {1,1,1}
        };
        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(allAlive);
        var onlyCornersAlive = new int[][]{
                {1,0,1},
                {0,0,0},
                {1,0,1}
        };
        assertArrayEquals(onlyCornersAlive, nextGeneration);
    }
}
```

It passes! 

We still have two scenarios from our original list that don't yet have tests that are worth considering, but first we are
going to make sure we haven't already covered them in one of our existing tests. Looking back through our tests we do
find that they are already handled. We could add more tests but I don't personally see the added value they would provide.
We also want to keep in mind that tests aren't free, they also have a maintenance cost associated with them.

With that I am going to wrap up this exercise. Thanks for reading!