---
title: "Conway's Game of Life: Refactoring"
last_modified_at: 2020-05-23
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
We have one of the four rules in the Game of Life working but, if we're honest, the code isn't great. Before we move on
to writing a test for another one of the Game of Life rules lets dive into the **refactor** phase since all of our tests
are currently passing.

Right now our solution looks like:
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
What makes this code difficult to understand? The of the variables names 'nx' and 'ny' aren't great. We also have the
4 nested loop monstrosity, at the bottom of which a temporary variable is changed. None of this is good, but the real
reason this code is difficult to understand is because our method is trying doing too much. It's both trying to count 
the number of alive neighbors and change the next generation state based of that count. Simple methods are easy to
understand, and simple methods are methods that do just a single thing.

Rather than refactor at the small scale and changing something simple like a variable name, we are actually going to 
first extract the alive neighbor counting logic into a dedicated method. We'll name this method something obvious like
`countAliveNeighbors` and provide the information necessary for it to accomplish the task.

```java
int[][] nextGeneration(int[][] grid) {
    for(int x = 0; x < grid.length; x++) {
        for(int y = 0; y < grid[x].length; y++) {
            int aliveNeighbors = countAliveNeighbors(grid, x, y);
            if(aliveNeighbors != 2) {
                grid[x][y] = 0;
            }
        }
    }
    return grid;
}
```

The results are immediate! While we still have the temporary variable `aliveNeighbors`, we are no longer  modifying it 
within our outer double for-loop. Extracting this logic into a dedicated method had a significant to the ease of 
understanding of the original method, and significantly less prone to error now that the `aliveNeighbors` has been 
removed.

Our new method 'countAliveNeighbors' is also very simple. The bad variable names still exist, but there are zero 
side-effects and it does exactly what it says on the tin, a great trait for any bit of code!

```java
private int countAliveNeighbors(int[][] grid, int x, int y) {
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
    return aliveNeighbors;
}
```

Next I'd like to remove the `continue` from the `countAliveNeighbors` method by inverting the if statement. I find the
specialized flow control statements like `continue`, `break`, etc difficult to reason about. They are essentially `goto`
statements, just restricted to a very narrow context. The `goto` statement has long been [considered harmful](https://www.google.com/search?q=goto+considered+harmful)
and these constructs should be considered similar in my opinion. Should you ever use them? Of course, but I prefer to
avoid them if possible as is the case here.

```java
private int countAliveNeighbors(int[][] grid, int x, int y) {
    int aliveNeighbors = 0;
    // Pivot around the cell at x,y, keeping track of how many live neighboring cells are found
    for(int nx = x - 1; nx < x + 2; nx++) {
        for(int ny = y - 1; ny < y + 2; ny++) {
            if(nx >= 0 && nx < grid.length && ny >= 0 && ny < grid[x].length) {
                if (nx != x || ny != y) {
                    aliveNeighbors += grid[nx][ny];
                }
            }
        }
    }
}
```

Next we are going to move the x coordinate boundary checks to be performed before worrying about the y coordinate. This 
is a bit of a lateral move, but is a slight improvement as we move verification logic to be immediately after the 
assignment of that variable, reducing the distance between the variable and its usage.

```java
private int countAliveNeighbors(int[][] grid, int x, int y) {
    int aliveNeighbors = 0;
    // Pivot around the cell at x,y, keeping track of how many live neighboring cells are found
    for(int nx = x - 1; nx < x + 2; nx++) {
        if(nx >= 0 && nx < grid.length) {
            for (int ny = y - 1; ny < y + 2; ny++) {
                if (ny >= 0 && ny < grid[x].length) {
                    if (nx != x || ny != y) {
                        aliveNeighbors += grid[nx][ny];
                    }
                }
            }
        }
    }
}
```

I'd also like to introduce an explicit 'isAlive' check. The behavior is currently is implicit, obscured by the value 
found at `grid[nx][ny]`. This may seem simple, but we're actually relying on an assumption that a "dead" cell is always
represented by a value of "0". I want these assumptions to be made explicit so, if necessary, we can easily find and
update them.

```java
private int countAliveNeighbors(int[][] grid, int x, int y) {
    int aliveNeighbors = 0;
    // Pivot around the cell at x,y, keeping track of how many live neighboring cells are found
    for(int nx = x - 1; nx < x + 2; nx++) {
        if(nx >= 0 && nx < grid.length) {
            for (int ny = y - 1; ny < y + 2; ny++) {
                if (ny >= 0 && ny < grid[x].length) {
                    if (nx != x || ny != y) {
                        aliveNeighbors += grid[nx][ny];
                        if(grid[nx][ny] == 1) {
                            aliveNeighbors++;
                        }
                    }
                }
            }
        }
    }
}
```

With the liveness check made explicit, let's take it a step further and introduce a method.

```java
private boolean isAlive(int [][] grid, int x, int y) {
    return grid[x][y] == 1;
}
```

And use it in `countAliveNeighbors`.

```java
private int countAliveNeighbors(int[][] grid, int x, int y) {
    int aliveNeighbors = 0;
    // Pivot around the cell at x,y, keeping track of how many live neighboring cells are found
    for(int nx = x - 1; nx < x + 2; nx++) {
        if(nx >= 0 && nx < grid.length) {
            for (int ny = y - 1; ny < y + 2; ny++) {
                if (ny >= 0 && ny < grid[x].length) {
                    if (nx != x || ny != y) {
                        aliveNeighbors += grid[nx][ny];
                        if(isAlive(grid, nx, ny)) {
                            aliveNeighbors++;
                        }
                    }
                }
            }
        }
    }
}
```

Another assumption we've been making is that any cell outside the bounds of our grid is implicitly dead. Now that we
have a dedicated method for reporting on the alive state of any given (x,Y) coordinate we should move the array bounds
checks into there.

```java
private boolean isAlive(int [][] grid, int x, int y) {
    if(x >= 0 && x < grid.length && y >= 0 && y < grid[x].length) {
        return grid[x][y] == 1;
    }
    return false;
}
```

Now all of our assumptions about what constitutes an alive (or dead) cell is centralized into a single, simple method!
This also allows us to remove the array boundary checks from `countAliveNeighbors`!

```java
private int countAliveNeighbors(int[][] grid, int x, int y) {
    int aliveNeighbors = 0;
    // Pivot around the cell at x,y, keeping track of how many live neighboring cells are found
    for(int nx = x - 1; nx < x + 2; nx++) {
        for (int ny = y - 1; ny < y + 2; ny++) {
            if (nx != x || ny != y) {
                aliveNeighbors += grid[nx][ny];
                if(isAlive(grid, nx, ny)) {
                    aliveNeighbors++;
                }
            }
        }
    }
}
```

Lets focus on the for loops themselves now. Considering that each cell has a finite number of neighbors (8) I feel using
a for loops actually hides the intent. What if, instead, we unrolled the loop? Thinking back to the 
[Transformation Priority Premise](http://blog.cleancoder.com/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html),
if we want to simplify our code we should consider how to trace our way back **up** the transformations, and a statement
is simpler than a loop. Since we only have eight neighbors to visit, I argue that eight explicit statements to visit 
each neighbor will be cleaner than the nested for-loop.

To perform the refactoring we'll start with the outer-most for loop by commenting out the line that increments the 
`'aliveNeighbor` variable. Next we'll duplicate the line `aliveNeighbors += isAlive(grid, nx, ny) ? 1: 0` 3 times, 
substituting `ny` for the 3 possible values (`y-1`, `y`, and `y+1`). Finally, we don't want to consider the pivot cell, 
so we'll place a guard clause around the potential (x,y) condition.

```java
private int countAliveNeighbors(int[][] grid, int x, int y) {
    int aliveNeighbors = 0;
    // Pivot around the cell at x,y, keeping track of how many live neighboring cells are found
    for(int nx = x - 1; nx < x + 2; nx++) {
        aliveNeighbors += isAlive(grid, nx, y - 1) ? 1 : 0;
        if(nx != x) {
            aliveNeighbors += isAlive(grid, nx, y) ? 1 : 0;
        }
        aliveNeighbors += isAlive(grid, nx, y + 1) ? 1 : 0;
        for (int ny = y - 1; ny < y + 2; ny++) {
            if (nx != x || ny != y) {
                aliveNeighbors += grid[nx][ny];
                if(isAlive(grid, nx, ny)) {
                    //aliveNeighbors++;
                }
            }
        }
    }
}
```

The code is actually worse! But sometimes it is necessary to take a few steps backward in order to move even further
forward. With our tests still passing (!), I feel comfortable removing the inner for-loop. We're left with:

```java
private int countAliveNeighbors(int[][] grid, int x, int y) {
    int aliveNeighbors = 0;
    // Pivot around the cell at x,y, keeping track of how many live neighboring cells are found
    for(int nx = x - 1; nx < x + 2; nx++) {
        aliveNeighbors += isAlive(grid, nx, y - 1) ? 1 : 0;
        if(nx != x) {
            aliveNeighbors += isAlive(grid, nx, y) ? 1 : 0;
        }
        aliveNeighbors += isAlive(grid, nx, y + 1) ? 1 : 0;
    }
}
```

Now for the outer for-loop. Again, we're going unroll the loop and comment out our anywhere we currently increment the 
`aliveNeighbors` count.

```java
private int countAliveNeighbors(int[][] grid, int x, int y) {
        int aliveNeighbors = 0;
        // Pivot around the cell at x,y, keeping track of how many live neighboring cells are found

        aliveNeighbors += isAlive(grid, x - 1, y - 1) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x - 1, y) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x - 1, y + 1) ? 1 : 0;

        aliveNeighbors += isAlive(grid, x, y - 1) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x, y + 1) ? 1 : 0;

        aliveNeighbors += isAlive(grid, x + 1, y - 1) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x + 1, y) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x + 1, y + 1) ? 1 : 0;

        for(int nx = x - 1; nx < x + 2; nx++) {
//            aliveNeighbors += isAlive(grid, nx, y - 1) ? 1 : 0;
//            if(nx != x) {
//                aliveNeighbors += isAlive(grid, nx, y) ? 1 : 0;
//            }
//            aliveNeighbors += isAlive(grid, nx, y + 1) ? 1 : 0;
        }
        return aliveNeighbors;
    }
```

The tests pass! Time to remove the dead code!

```java
private int countAliveNeighbors(int[][] grid, int x, int y) {
        int aliveNeighbors = 0;

        aliveNeighbors += isAlive(grid, x - 1, y - 1) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x - 1, y) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x - 1, y + 1) ? 1 : 0;

        aliveNeighbors += isAlive(grid, x, y - 1) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x, y + 1) ? 1 : 0;

        aliveNeighbors += isAlive(grid, x + 1, y - 1) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x + 1, y) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x + 1, y + 1) ? 1 : 0;

        return aliveNeighbors;
    }
```

I'm now happy with where the code is. It's much easier to follow, and I think we've established a solid foundation on
which to continue building. I don't love repeating the ternary 8 times, but I'm OK with the tradeoff for now. That may
change, however. The full (but still incomplete!) solution is below.

```java
class GameOfLife {
    int[][] nextGeneration(int[][] grid) {
        for(int x = 0; x < grid.length; x++) {
            for(int y = 0; y < grid[x].length; y++) {
                int aliveNeighbors = countAliveNeighbors(grid, x, y);
                if(aliveNeighbors != 2) {
                    grid[x][y] = 0;
                }
            }
        }
        return grid;
    }

    private int countAliveNeighbors(int[][] grid, int x, int y) {
        int aliveNeighbors = 0;

        aliveNeighbors += isAlive(grid, x - 1, y - 1) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x - 1, y) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x - 1, y + 1) ? 1 : 0;

        aliveNeighbors += isAlive(grid, x, y - 1) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x, y + 1) ? 1 : 0;

        aliveNeighbors += isAlive(grid, x + 1, y - 1) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x + 1, y) ? 1 : 0;
        aliveNeighbors += isAlive(grid, x + 1, y + 1) ? 1 : 0;
        
        return aliveNeighbors;
    }

    private boolean isAlive(int [][] grid, int x, int y) {
        if(x >= 0 && x < grid.length && y >= 0 && y < grid[x].length) {
            return grid[x][y] == 1;
        }
        return false;
    }
}
```