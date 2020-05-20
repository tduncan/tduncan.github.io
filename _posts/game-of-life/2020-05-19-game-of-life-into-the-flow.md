---
title: "Conway's Game of Life: Into the Flow"
last_modified_at: 2020-05-19
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
We last left of with our first failing test.

```
org.opentest4j.AssertionFailedError: actual array was <null>
	at io.github.tduncan.gameoflife.GameOfLifeTest.gridWithAllDeadCellsResultsInAllCellsRemainingDead(GameOfLifeTest.java:20)
```

We're not in the **red** phase of the red-green-refactor TDD cycle, and our goal is to get into the **green** phase as
quickly as possible. How can we do that? It turns the answer is quite simple, but this is a bit of a trick question.

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        return new int[][]{
                {0, 0, 0},
                {0, 0, 0},
                {0, 0, 0}
        };
    }
}
```
We just need to replace our `null` return with a hardcoded two-dimensional array containing only dead cells! Now this 
is clearly not complete solution, but it doesn't have to be! At least not yet. This is how our tests **drive** the
development. Our implementation is "complete" when all of our tests pass. If our current implementation isn't good
enough, we will add additional tests that force the implementation to reach accordingly. Don't take this to mean we are
testing the implementation because we aren't. We are testing the **behavior** through the public APIs, not specifically
how that behavior is achieved.

With our tests now passing we have entered the **refactor** phase. We're really early here, though, so I don't yet see
an opportunity to obviously improve the code. Instead we'll move onto our next test. First, however, I want to add a
couple of test cases to our list that I think will help us nudge our solution forward to something more general.

```    
// all cells are dead, but grid is not 3x3
// static grid, but some cells are alive
-----
// all cells are alive, only corners remain alive
// live cell with zero live neighbors should die
// live cell with one live neighbor, should die
// live cell with two live neighbors, should live
// live cell with three live neighbors, should live
// live cell with four live neighbors, should die
// dead cell with one live neighbor, should remain dead
// deal cell with two live neighbors, should remain dead
// dead cell with three live neighbors, should be alive
// dead cell with four live neighbors, should be dead
```
These new cases were directly inspired by how we passed our first test. It's always a good idea to keep a running
todo list for exactly this reason.

Now, how do we decide what test comes next? Just as with our implementation, we want to slowly build up our tests from
the simple test cases to the more complex. Given that we have a test already, a good heuristic for deciding on the next
test is to copy the previous test, then make a minor change that will result in the test failing. Building from our
first test, and consulting our list of test cases, I think our next case is `all cells are dead, but grid is not 3x3`.
So let's translate that into a test.

```java
class GameOfLifeTest {
    ...

    @Test
    void largerGridWithAllDeadCellsResultsInAllCellsRemainingDead() {
        var grid = new int[][] {
                {0,0,0,0},
                {0,0,0,0},
                {0,0,0,0},
                {0,0,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var allDead = new int[][]{
                {0,0,0,0},
                {0,0,0,0},
                {0,0,0,0},
                {0,0,0,0}
        };
        assertArrayEquals(allDead, nextGeneration);
    }
}
```
The only change we've made is to increase the size of the grid from a 3x3 to a 4x4 grid. It's a small change, but will
force our solution to stop making assumptions about the size of the grid and toward a slightly more generic solution.
Running the tests (don't forget to run all the tests!) and we see the following failure:

```
org.opentest4j.AssertionFailedError: array lengths differ, expected: <4> but was: <3>
	at io.github.tduncan.gameoflife.GameOfLifeTest.largerGridWithAllDeadCellsResultsInAllCellsRemainingDead(GameOfLifeTest.java:36)
```

Passing this is easy enough. We just need to return a two-dimensional array of all dead cells that has the same
dimensions as the provided grid. Remember, our goal is to keep all of our tests passing from here on out.

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        return new int[grid.length][grid[0].length];
    }
}
```

Passed! A very simple, but important, step toward a general solution. For our next test we are going to go back to a 
3x3 grid, but we need to think of a 3x3 grid that will remain static from one generation to the next and which will 
force our solution away the assumption the cells are always dead.

```java
class GameOfLifeTest {
    ...

    @Test
    void gridContainingAliveCellsAndRemainsTheSameInTheNextGeneration() {
        var grid = new int[][]{
                {1,1,0},
                {1,1,0},
                {0,0,0}
        };

        var game = new GameOfLife();
        var nextGeneration = game.nextGeneration(grid);

        var unchangedGrid = new int[][]{
                {1,1,0},
                {1,1,0},
                {0,0,0}
        };
        assertArrayEquals(unchangedGrid, nextGeneration);
    }
}
```

Here we have one of the known stable patterns known unimaginatively as "the block". Because each alive cell is
neighboring three other alive cells they will continue to live into the next generation. Conversely, none of the dead
cells neighbor exactly three alive cells guaranteeing they will continue to stay dead.

```
org.opentest4j.AssertionFailedError: array contents differ at index [0][0], expected: <1> but was: <0>
	at io.github.tduncan.gameoflife.GameOfLifeTest.gridContainingAliveCellsAndRemainsTheSameInTheNextGeneration(GameOfLifeTest.java:50)
```

The test fails! Perfect! This will now force our solution away from the assumption that all the grid cells should be
dead in the array that is returned. To pass the test we're going to use a trick. We're actually going to return echo
the input as the output!

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        return grid;
    }
}
```

This may feel like cheating and a waste of time, but there's actually an important advancement hidden in this step. 
First, we are no longer hardcoding either the state of the individual grid cells or the size of the grid.  Second, the 
computed next generation is now derived from the input! This is a big deal.

We've also just completed one of the first and most basic steps along what Uncle Bob has termed the [The Transformation 
Priority Premise](http://blog.cleancoder.com/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html). We won't go 
into too much detail here, but this insight by Uncle Bob is a framework we will continue to utilize when making
decisions about what test to write next as we try to drive our Game of Life implementation toward a generic solution.

With both of our tests passing we'll again reference our todo list, the next one up is `All cells are alive`, so let's 
start writing that test.

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

```
org.opentest4j.AssertionFailedError: array contents differ at index [0][1], expected: <0> but was: <1>
	at io.github.tduncan.gameoflife.GameOfLifeTes.gridContainAllLiveCellsWillOnlyHaveCornersRemainingAlive(GameOfLifeTest.java:69)
```
Our next failing test! And our first scenario that requires a state change from one generation to the next. This test
raises some interesting things to discuss, which we'll do in the next post.