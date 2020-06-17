---
title: "Digging A Hole"
permalink: /tdd/examples/conways-game-of-life/digging-a-hole
last_modified_at: 2020-06-16
---
Our Game of Life solution is now easier to understand so let's revisit our list of use cases and continue driving 
toward a complete implementation. Looking at our notes I think the scenario *live cell with one live neighbor, 
should die* is an OK place to go next. This seems like a reasonably simple scenario so let's start converting it into a 
test.

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

The test passes... Why?

Ideally all of our tests will fail at first, but this is a completely normal situation in which to find yourself so 
don't worry.

When this happens, take a step back to understand why? Once you understand why the test that we expected to fail
actually passes try to think of a scenario that can be converted into a failing test. At the same time you will want to
disable the passing test to avoid the possibility that it starts failing as you focus on the new test.

If you can't think of one, congrats! You may be done! Otherwise, start writing that new test. For us, that test will be 
a scenario that causes a dead cell to be alive in the next generation. Why? Because currently our solution only changes
cells to dead in the next generation.

In our notes we jotted down "dead cell with three live neighbors, should be alive", so we'll start there.

```java
class GameOfLifeTest {
    @Test
    void deadCellWithThreeLiveNeighborsShouldBecomeAliveInNextGeneration() {
        var grid = new int[][] {
                {0,1,0},
                {1,0,1},
                {0,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var expectedNextGeneration = new int[][] {
                {0,1,0},
                {0,1,0},
                {0,0,0}
        };
        assertArrayEquals(expectedNextGeneration, nextGeneration);
    }
}
```
```
org.opentest4j.AssertionFailedError: array contents differ at index [1][1], expected: <1> but was: <0>
	at io.github.tduncan.gameoflife.GameOfLifeTest.deadCellWithThreeLiveNeighborsShouldBecomeAliveInNextGeneration(GameOfLifeTest.java:127)
```

We have our failing test. We now must implement the 4th rule of the Game of Life *Any dead cell with exactly three live 
neighbours becomes a live cell, as if by reproduction*.

To do this we are going to use the `isAlive` method that as a decision point. We are going to place our existing logic
within a block that will only execute when the current cell is alive, and we're going to introduce a block dedicated to
operate specifically on dead cells.

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        for(int x = 0; x < grid.length; x++) {
            for(int y = 0; y < grid[x].length; y++) {
                int aliveNeighbors = countAliveNeighbors(grid, x, y);

                if(isAlive(grid, x, y)) {
                    if (aliveNeighbors != 2) {
                        grid[x][y] = 0;
                    }
                } else {
                    if (aliveNeighbors == 3) {
                        grid[x][y] = 1;
                    }
                }
            }
        }
        return grid;
    }

    ...
}
```

This looks good. Let's run the tests.

```
org.opentest4j.AssertionFailedError: array contents differ at index [1][1], expected: <1> but was: <0>
	at io.github.tduncan.gameoflife.GameOfLifeTest.deadCellWithThreeLiveNeighborsShouldBecomeAliveInNextGeneration(GameOfLifeTest.java:127)

org.opentest4j.AssertionFailedError: array contents differ at index [1][1], expected: <0> but was: <1>
	at io.github.tduncan.gameoflife.GameOfLifeTest.liveCellWithTwoLiveNeighborsShouldContinueLivingInNextGeneration(GameOfLifeTest.java:108)
```

Two failing tests?!

This is a bad situation to find ourselves in. Let's take a moment to consider what's going on. Not only does our latest
test fail, but we've somehow also introduced a regression causing an existing test to fail. Below are the two failing
tests:

```java
class GameOfLifeTest {
    ...


    @Test
    void liveCellWithTwoLiveNeighborsShouldContinueLivingInNextGeneration() {
        var grid = new int[][] {
                {1,1,0},
                {1,0,0},
                {0,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var expectedNextGeneration = new int[][] {
                {1,1,0},
                {1,0,0},
                {0,0,0}
        };
        assertArrayEquals(expectedNextGeneration, nextGeneration);
    }

    @Test
    void deadCellWithThreeLiveNeighborsShouldBecomeAliveInNextGeneration() {
        var grid = new int[][] {
                {0,1,0},
                {1,0,1},
                {0,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var expectedNextGeneration = new int[][] {
                {0,1,0},
                {0,1,0},
                {0,0,0}
        };
        assertArrayEquals(expectedNextGeneration, nextGeneration);
    }
}
```

Taking a closer look I realize that both scenarios actually require more than 2 rules to be implemented. This can hard
to avoid, especially as the use cases become more involved. However, I think we made a mistake in
`liveCellWithTwoLiveNeighborsShouldContinueLivingInNextGeneration`, which was intended to be as simple as possible.

In order to move forward I only want to focus on a single failing test at a time. My plan is to disable our new test
`deadCellWithThreeLiveNeighborsShouldBecomeAliveInNextGeneration` and use a better input for
`liveCellWithTwoLiveNeighborsShouldContinueLivingInNextGeneration`. With our newest test disabled we are also going to 
revert our implementation back to the last known good state we can have confidence with our updated version of
`liveCellWithTwoLiveNeighborsShouldContinueLivingInNextGeneration`.

```java
class GameOfLifeTest {
    ...

    @Test
    void liveCellWithTwoLiveNeighborsShouldContinueLivingInNextGeneration() {
        var grid = new int[][] {
                {0,0,1},
                {0,1,0},
                {1,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var expectedNextGeneration = new int[][] {
                {0,0,0},
                {0,1,0},
                {0,0,0}
        };
        assertArrayEquals(expectedNextGeneration, nextGeneration);
    }
}
```

We've changed our starting generation to "pivot" around the center cell, and I've specifically chosen the upper-right
and lower-left cells to start alive so minimize the number of alive cells that will interact with each other. But,
running the tests results a bit of a surprise:

```
org.opentest4j.AssertionFailedError: array contents differ at index [1][1], expected: <1> but was: <0>
	at io.github.tduncan.gameoflife.GameOfLifeTest.liveCellWithTwoLiveNeighborsShouldContinueLivingInNextGeneration(GameOfLifeTest.java:108)
```

What's happening? Well, we actually have a bug in our solution! The Game of Life intends for each generation to be 
derived from the previous generation, but for each generation to be immutable. The term immutable should be a big clue 
as to what's going on here. We are actually modifying the current generation, not producing a new generation! Sometimes,
those modifications cause future cells evaluated at a later point in time to be assigned the wrong state!

So, before to get our tests back to passing we need to address this problem now.

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        int[][] nextGeneration = new int[grid.length][grid[0].length];
        for(int x = 0; x < grid.length; x++) {
            for(int y = 0; y < grid[x].length; y++) {
                int aliveNeighbors = countAliveNeighbors(grid, x, y);
                if (aliveNeighbors != 2) {
                    nextGeneration[x][y] = 0;
                } else {
                    nextGeneration[x][y] = grid[x][y];
                }
            }
        }
        return nextGeneration;
    }
    
    ...
}
```

Our generations are now immutable. But, if we run our tests we find a new failure...

```
org.opentest4j.AssertionFailedError: array contents differ at index [0][0], expected: <1> but was: <0>
	at io.github.tduncan.gameoflife.GameOfLifeTest.gridContainingAliveCellsAndRemainsTheSameInTheNextGeneration(GameOfLifeTest.java:51)
```

Before you become discouraged, take comfort in knowing that the test is correct, but our solution is no longer good 
enough! We actually need to implement another rule that we were able to cheat in the past, which is that a live cell 
should remain alive when it has three live neighbors. So let's do that!

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        int[][] nextGeneration = new int[grid.length][grid[0].length];
        for(int x = 0; x < grid.length; x++) {
            for(int y = 0; y < grid[x].length; y++) {
                int aliveNeighbors = countAliveNeighbors(grid, x, y);
                if (aliveNeighbors != 2 && aliveNeighbors != 3) {
                    nextGeneration[x][y] = 0;
                } else {
                    nextGeneration[x][y] = grid[x][y];
                }
            }
        }
        return nextGeneration;
    }

    ...
}
```

All tests passing! It finally feels like we're getting back on track. Let's revisit the test that started us down this
rabbit hole.

```java
class GameOfLifeTest {
    ...

    @Test
    void deadCellWithThreeLiveNeighborsShouldBecomeAliveInNextGeneration() {
        var grid = new int[][] {
                {0,1,0},
                {1,0,1},
                {0,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var expectedNextGeneration = new int[][] {
                {0,1,0},
                {0,1,0},
                {0,0,0}
        };
        assertArrayEquals(expectedNextGeneration, nextGeneration);
    }
}
```

```
org.opentest4j.AssertionFailedError: array contents differ at index [1][1], expected: <1> but was: <0>
	at io.github.tduncan.gameoflife.GameOfLifeTest.deadCellWithThreeLiveNeighborsShouldBecomeAliveInNextGeneration(GameOfLifeTest.java:127)
```

A single failing test! And it's the one we want. We're definitely back on track! Let's try re-introducing our decision
point around whether the current cell is alive.

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        int[][] nextGeneration = new int[grid.length][grid[0].length];
        for(int x = 0; x < grid.length; x++) {
            for(int y = 0; y < grid[x].length; y++) {
                int aliveNeighbors = countAliveNeighbors(grid, x, y);

                if(isAlive(grid, x, y)) {
                    if (aliveNeighbors != 2 && aliveNeighbors != 3) {
                        nextGeneration[x][y] = 0;
                    } else {
                        nextGeneration[x][y] = grid[x][y];
                    }
                } else {
                    if(aliveNeighbors == 3) {
                        nextGeneration[x][y] = 1;
                    }
                }
            }
        }
        return nextGeneration;
    }

    ...
}
```

All of our tests are passing!

That was a bit of a winding journey, but we ultimately found our way. Notice the role our tests played in helping guide
us to the right place. They weren't the only tool, you always have to think your way through the problem, but due to the
tests describing and asserting on the externally observable behavior of our solution we provided ourselves with an
executable specification with clear examples of inputs and expected outputs.

This adventure was a great example of why the health of your tests are a critical component that needs to be paid just
as much attention (likely more if we're being honest) as the code being tested.