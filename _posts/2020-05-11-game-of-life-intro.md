---
title: "Conway's Game of Life: Introduction"
last_modified_at: 2020-05-11
categories:
  - Game Of Life
tags:
  - TDD
  - TDD By Example
  - Game of Life
  - Java
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
Conway's Game of Life is not actually a game, at least not in the common use of the word, but rather a simulation.
More formally, the Game of Life is classified as a cellular automation, of which it is perhaps the most well-known.
The straightforward solution is quite short, yet there is just enough complexity that I still need to think my way
through certain parts, even after solving it a number of times.  It's my favorite yet non-trivial "toy" problem.

Really, though, the "game" is super simple. The simulation takes place in a two-dimensional grid of cells, and each
cell has a state of either alive or dead. The simulation then advances through generations, and the state of a cell in 
each new generation is independently derived from its previous state and the collective state of that of the neighboring 
cells.

How are the cell states computed? There are 4 rules:
1. Any live cell with fewer than two live neighbours dies, as if by underpopulation.
2. Any live cell with two or three live neighbours lives on to the next generation.
3. Any live cell with more than three live neighbours dies, as if by overpopulation.
4. Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.

Let's use an example to demonstrate how the algorithm works. We'll assume that anything outside of our 3x3 grid is dead.

```
1st Gen              2nd Gen              3rd Gen
+---+---+---+        +---+---+---+        +---+---+---+
|   | * | * |        |   | * |   |        |   |   |   |
+---+---+---+        +---+---+---+        +---+---+---+
| * |   |   |  --->  |   |   | * |  --->  |   |   |   |
+---+---+---+        +---+---+---+        +---+---+---+
|   |   | * |        |   |   |   |        |   |   |   |
+---+---+---+        +---+---+---+        +---+---+---+
```
The top-left cell starts dead and continues in the dead state for the 2nd and 3rd generations. This is because it only
has two neighbors that are alive and doesn't qualify for rule #4. The top-middle cell starts alive, and continues
living into the 2nd generation as it qualifies for rule #2. However, when moving into the 3rd generation it qualifies
for rule #1 (underpopulation) and dies. Advancing a little, take a look at the middle-right cell. It starts out as dead,
but because it has three alive neighbors in the 1st generation is becomes alive in the 2nd generation due to rule #4.
It suffers the safe fate as the top-middle cell in the 3rd generation, though, and quickly dies out. In the end, all of
the cells die and will continue to stay dead under this example.

Most examples will end this way. Over the years a number of configurations have been identified as either stable or
self-replicating. Takes the "book", for example. As long as the surrounding cells remain dead the "book" will
continue on unchanged.

```
1st Gen                  2nd Gen                  3rd Gen
+---+---+---+---+        +---+---+---+---+        +---+---+---+---+
|   |   |   |   |        |   |   |   |   |        |   |   |   |   |
+---+---+---+---|        +---+---+---+---|        +---+---+---+---+
|   | * | * |   |  --->  |   | * | * |   |  --->  |   | * | * |   |
+---+---+---+---|        +---+---+---+---|        +---+---+---+---|
|   | * | * |   |        |   | * | * |   |        |   | * | * |   |
+---+---+---+---}        +---+---+---+---|        +---+---+---+---+
|   |   |   |   |        |   |   |   |   |        |   |   |   |   |
+---+---+---+---+        +---+---+---+---+        +---+---+---+---+
```

A simple self-replicating example is the "blinker". Starting as either a 3-cell horizontal or vertical line, the
shape of the object will oscillate between the two configurations indefinitely if left alone.
```
1st Gen              2nd Gen              3rd Gen
+---+---+---+        +---+---+---+        +---+---+---+       +---+---+---+
|   | * |   |        |   |   |   |        |   | * |   |       |   |   |   |
+---+---+---+        +---+---+---+        +---+---+---+       +---+---+---+
|   | * |   |  --->  | * | * | * |  --->  |   | * |   | --->  | * | * | * |
+---+---+---+        +---+---+---+        +---+---+---+       +---+---+---+
|   | * |   |        |   |   |   |        |   | * |   |       |   |   |   |
+---+---+---+        +---+---+---+        +---+---+---+       +---+---+---+
```
There are even self-replicating that have been discovered that move across the grid call spaceships! All of this organic
behavior is formed out of just the 4 simple rules laid out above. Super cool!

Conway's Game of Life is deceptively deep! We could go on forever, but what I'm really hoping to do is use the problem
as a demonstration of Test Driven Development, which is exactly what we'll do in the next post.