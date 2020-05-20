---
title: "Conway's Game of Life: The First Test"
last_modified_at: 2020-05-19
categories:
  - TDD Examples
  - Game Of Life
tags:
  - TDD
  - Java
  - JUnit 5
version: 1.0.2
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
We'll start at the beginning of every Java program, the boilerplate. Right now this is test class itself, but we're
also going to copy our list of test cases over to act as a running todo list.

```java
class GameOfLifeTest {
    // all cells are dead, should all remain dead
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
}	    
```

Simple enough! Now for our first test, and our first interesting discussion: test naming!

We need to translate our first listed test case into a JUnit 5 test, which means we need to create a method to enclose
the scenario. How we name our method is very important. At the same time, a test method name is perhaps the safest and
most straightforward change that can be made so don't stress too much if the name is perfect. It can be changed! And
easily!

Test names are important, though. Our goal is to accurately describe the scenario, ideally including information about
the pre-conditions and the expected result. At the same time we want the language to be at the level of the domain and
avoid dropping into implementation specific details (think method name, return codes, etc). It's fine for test names to
be long, but also keep in mind that the test itself will help in telling the story. In the end I chose to use
`gridWithAllDeadCellsResultsInAllCellsRemainingDead` which I felt did an OK job describing both the initial and expected
states of the system. Translating this into a JUnit 5 test is just a matter of annotating the new method with `@Test`.

```java
import org.junit.jupiter.api.Test;

class GameOfLifeTest {
    @Test
    void gridWithAllDeadCellsResultsInAllCellsRemainingDead() {
    }
}
```

Now to begin describing our scenario in code. We're going to start very simple and see where it takes us. The Game of
Life describes the universe as a *two-dimensional grid of cells*, so we're going to start as simple as we can by
literally defining the grid as a two-dimensional array. The cells themselves will be integers. 

```java
class GameOfLifeTest {
    @Test
    void gridWithAllDeadCellsResultsInAllCellsRemainingDead() {
        var grid = new int[][] {
                {0,0,0},
                {0,0,0},
                {0,0,0}
        };
        
        var game = new GameOfLife();
    }
}
```

One school of TDD thought advocates to literally not write any production code, including the production class, 
until a failing test requires it, and that a compiler error is a failing test.  I've done that here for demonstration, 
but I personally don't find that level of rigor valuable.  I strongly advise against creating method stubs, however, as 
exploring the API space within the tests from the perspective of a user can produce some surprising results. Often the 
APIs we dream up are not very usable, and this pain will be felt frequently as you continue building the test suite.
Don't ignore that pain, it is a strong signal the design needs to be improved. If it hurts you when you have the full
context of the problem space, just imagine how annoying it will be for someone else, possibly even your future self, 
trying to integrate with the component later on.

So we create the `GameOfLife` class to satisfy the compiler, but nothing else. Again, we want to explore the API space 
from the perspective of a user.

```java
package io.github.tduncan.gameoflife;

class GameOfLife {
}
```

Our requirements are fairly simple for the Game of Life, however we still need to make the decision about how the class
should be used. We've already decided we want to operate on a two-dimensional array of integers, but how to provide it?
The two obvious options are as a method parameter or as a constructor parameter. If we were to choose the constructor
parameter option we would be required to manage the state explicitly as well as provide some mechanism for an outside
observer to inspect that state (the tests in our case). That seems more complicated than should be necessary, so I opt 
to supply the grid as a method parameter, and for the next generation to be returned as another two-dimensional grid 
back. This solves both the state management and observability problem.

The only question that remains is how to name our method. Just as before, don't stress too much on finding the perfect 
name, particularly not when you are just building up the solution. Just look for a name that clearly expresses what the
method does. A method name is the easiest change to make, and any decent IDE will have this as an automated refactoring 
built in. Once the component is in use renaming is still generally a very safe change. Do take more care, however, if 
the method in is part of the public facing API of a library.

The name I settle on is `nextGeneration`, which will accept the two-dimensional array as input and return another
two-dimensional array as the output. 

```java
class GameOfLifeTest {
    @Test
    void gridWithAllDeadCellsResultsInAllCellsRemainingDead() {
        var grid = new int[][] {
                {0,0,0},
                {0,0,0},
                {0,0,0}
        };
        
        var game = new GameOfLife();
        int[][] nextGeneration = game.nextGeneration(grid);
    }
}
```

Our method doesn't exist yet, so as before with the `GameOfLife` class we need to create it to satisfy the compiler.
Since we need to return something, again to satisfy the compiler, we will just return `null` so we can get back to our
test.

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        return null;
    }
}
```

To finish our test we need to fill verify our expectation, which we'll do using the `assertArrayEquals` static method
found on the `Assertions` class that JUnit 5 provides.

```java
import static org.junit.jupiter.api.Assertions.assertArrayEquals;

...

class GameOfLifeTest {
    @Test
    void gridWithAllDeadCellsResultsInAllCellsRemainingDead() {
        var grid = new int[][] {
                {0,0,0},
                {0,0,0},
                {0,0,0}
        };
        
        var game = new GameOfLife();
        int[][] nextGeneration = game.nextGeneration(grid);

        var allDead = new int[][] {
                {0,0,0},
                {0,0,0},
                {0,0,0}
        };
        assertArrayEquals(allDead, nextGeneration);
    }
}
```

Given that we stubbed our `nextGeneration` method out to return null we should now have our first failing test. Do
notice the `allDead` variable created, however. I've done this to make it more clear exactly what the expectation is,
including giving it a meaningful name. You may argue that the variable is unnecessary because we've already defined
this exact value for our initial state, but the fact that the two states are the same does not mean they are duplicate
of each other . It's easy to be tempted to take the shortcut and have the assertion be 
`assertArrayEquals(grid, nextGeneration);`, but doing so prevents our tests from telling the story accurately,
misleading the reader in some subtle and not-so-subtle ways.

Moving on, we expect the test to fail so lets run it to verify.

```
$ ./gradlew check
Starting a Gradle Daemon (subsequent builds will be faster)

> Task :test FAILED

GameOfLifeTest > gridWithAllDeadCellsResultsInAllCellsRemainingDead() FAILED
    org.opentest4j.AssertionFailedError at GameOfLifeTest.java:20

1 test completed, 1 failed
```

It fails! Perfect! In the next post we'll begin to flesh out our Game of Life solution and really get into the
red-green-refactor workflow that is the hallmark of TDD.