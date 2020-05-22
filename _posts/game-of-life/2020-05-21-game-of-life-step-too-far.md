---
title: "Conway's Game of Life: A Step Too Far"
last_modified_at: 2020-05-22
categories:
  - TDD Examples
  - Game Of Life
tags:
  - TDD
  - Java
  - JUnit 5
version: 1.0.0
author:
  name   : "Thomas Duncan"
  avatar : "/assets/images/photo.jpg"
  bio    : "Software Engineer.  TDD Enthusiast."
  location: "San Francisco Bay Area"
  links:
    - label: "Website"
      icon: "fas fa-fw fa-link"
      url: "https://tduncan.github.io/"
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/tduncan"
    - label: "Source Code"
      icon: "fas fa-fw fa-code"
      url: "https://github.com/tduncan/conways-game-of-life"
---
Picking back up where we left off, we have a failing test for the following scenario.

```
1st Gen              2nd Gen      
+---+---+---+        +---+---+---+
| * | * | * |        | * |   | * |
+---+---+---+        +---+---+---+
| * | * | * |  --->  |   |   |   |
+---+---+---+        +---+---+---+
| * | * | * |        | * |   | * |
+---+---+---+        +---+---+---+
```
Let's explore what our Game of Life solution will need to do in order to pass this test. The obvious difference between
this scenario and our previous test cases is that the expected next generation includes cells whose state has changed
from alive to dead, so we need to detect this. Specifically we need to apply the rule 3; *any live cell with more 
than three live neighbours dies, as if by overpopulation*. Perhaps less obvious, however, is the need to detect when 
an alive cell should remain alive in the next generation, or rule 2; *any live cell with two or three live neighbours 
lives on to the next generation*.

Once I realized this I had paused. We're trying to talk small steps forward, both with the implementation and the tests.
Our last test, though, is asking us to implement 2 of the 4 rules almost immediately. Ideally our test would have asked
us to implement just one of the rules, with the 2nd rule being enforeced with a dedicated test. We could continue down
our current path, but we run the potential of laying a trap for our future selves in the form of false-positive test
results. How? Imagine we test the other 2 rules with dedicated tests but do not add dedicated tests for the two being
tested by the scenario. It would then be possible for an incorrect implementation to pass all of our tests cases, but
not actually compute the next generation correctly. Eventually we'd realize the mistake and add a test to enforce the
desired behavior. Hopefully that realization would come before any real harm was done, but at best we've wasted our own
time versus initially writing tests dedicated to each rule.

We're going to disable the test for now, but we'll revisit it in the future.

```java
class GameOfLifeTest {
    ...

    @Test
    @Disabled
    void gridContainAllLiveCellsWillOnlyHaveCornersRemainingAlive() {
        ...
    }
}
```

Going back to our list of test cases I notice *live cell with zero live neighbors should die*, which seems like a 
reasonable place to go next. To keep things simple we are going to use 3x3 grids, generally using the center cell as
our 'pivot' point.

```java
class GameOfLifeTest {
    ...

    @Test
    void gridWithOnlySingleLiveCellShouldDieOutEntirely() {
        var grid = new int[][] {
                {0,0,0},
                {0,1,0},
                {0,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var allDead = new int[][]{
                {0,0,0},
                {0,0,0},
                {0,0,0}
        };
        assertArrayEquals(allDead, nextGeneration);
    }
}
```
```
org.opentest4j.AssertionFailedError: array contents differ at index [1][1], expected: <0> but was: <1>
	at io.github.tduncan.gameoflife.GameOfLifeTest.gridWithOnlySingleLiveCellShouldDieOutEntirely(GameOfLifeTest.java:70)
```

Now we are clearly requiring only the *underpopulation* rule to be implemented, at least partially. This should be a 
better place to start building our real solution to the Game of Life.

To get back into **green** quickly your first instinct is probably to start iterating through the two-dimensional grid,
counting neighbors, and fully applying the *underpopulation* rule, right? But doing so is doing more than is necessary
at this point in time. Instead we can take advantage of the fact that all of our existing tests have the cell located 
at (1,1) dead in the next generation. If we compute the coordinates of the "middle" cell and force it into a dead state
that should be enough to get to the next step. Let's try it!

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        int xMiddle = grid.length / 2;
        int yMiddle = grid[0].length / 2;
        grid[xMiddle][yMiddle] = 0;
        return grid;
    }
}
```

It passes! If you weren't rolling your eyes before you probably are now! Why am I wasting time with code like this? We 
want to get to **green** as fast as we can, but once there we also want to refactor toward the simplest bit of code 
necessary to pass all the tests. This part of the workflow, the **refactor** stage, is key. Don't skip it! Refactoring 
applies a downward pressure on complexity helping keep our code simple. It also applies an upward pressure on the tests 
themselves. If I can refactor the code into an incorrect implementation we obviously need a new test.

As for that next test? As always our tests should push our implementation closer and closer to a generic solution. And
just as before we should try to build our next test by building off of our previous test. Since our previous test
allowed us to make a bad assumption in code the next test should require us to remove that assumption.

```java
class GameOfLifeTest {
    @Test
    void anotherGridWithOnlySingleLiveCellShouldDieOutEntirely() {
        var grid = new int[][] {
                {0,1,0},
                {0,0,0},
                {0,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var allDead = new int[][]{
                {0,0,0},
                {0,0,0},
                {0,0,0}
        };
        assertArrayEquals(allDead, nextGeneration);
    }
}
```

This is the same scenario as before, however, we've changed the alive cell from (1,1) to (0,1).

```
org.opentest4j.AssertionFailedError: array contents differ at index [0][1], expected: <0> but was: <1>
    at io.github.tduncan.gameoflife.GameOfLifeTest.anotherGridWithOnlySingleLiveCellShouldDieOutEntirely(GameOfLifeTest.java:89)
```

Now we're forced to write some real code!

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        for(int x = 0; x < grid.length; x++) {
            for(int y = 0; y < grid[x].length; y++) {

                int aliveNeighbors = 0;
                // Pivot around the cell at x,y, keeping track of how many live neighboring cells are found
                for(int nx = x - 1; nx < x + 2; nx++) {
                    for(int ny = y - 1; ny < y + 2; ny++) {
                        if(nx >= 0 && nx < grid.length && ny >= 0 && ny < grid[x].length) {
                            if(nx == x && ny == y) {  // Don't count the pivot point
                                continue;
                            }
                            aliveNeighbors += grid[nx][ny];
                        }
                    }
                }

                if(aliveNeighbors < 1) {
                    grid[x][y] = 0;
                }
            }
        }
        return grid;
    }
}
```

It's not a lot of code, but it's certainly difficult to reason about! We're iterating through the two-dimensional grid,
then for each individual cell we are adding the number of alive neighbors. Finally, when the number of alive neighbors
is less than 1 we mark the cell as dead in the next generation. That sounds fairly straightforward, but the code is
certainly not. I was even tripped up a bit in the neighbor counting part, forgetting to skip the current cell and mixing
up the usage of `x` and `nx` or `y` and `ny` on occasion. These are all clear signals that we need to spend some time
refactoring, and that's exactly where we'll start next time.