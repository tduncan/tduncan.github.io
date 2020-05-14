---
title: "Conway's Game of Life: Getting Started"
last_modified_at: 2020-05-13
categories:
  - TDD Examples
  - Game Of Life
tags:
  - TDD
  - Java
  - JUnit 5
version: 1.0.1
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
Before we jump head first into the code we first need to come up with a plan for how we are going to move forward. 
Lame, I know, but planning is essential. We should categorically reject long, extended up front planning as inefficient, 
both the goal and a plan are essential ingredients of success.

I like to think of this step like planning a vacation. The
destination is the goal, but you also need to think about
transportation, meals, and lodging at the very least. You likely 
won't plan each of these in extreme detail, for example you may
choose a hotel with plenty of restaurants within walking distance,
but you still needed to spend some time locating the hotel. Failing
to plan at least this much of the vacation may find yourself in a
hotel without nearby restaurants, or without one altogether!

So, we want to plan, but not too much! Just enough to get us going.
Our expectation is that by working through the problem we will
discover nuances that weren't clear before. Every step of the way
we will reference our plan and adjust it as necessary with our 
final goal in mind.

With that, let's think back to the rules of Conway's Game of Life.
> 1. Any live cell with fewer than two live neighbours dies, as if by underpopulation.
> 2. Any live cell with two or three live neighbours lives on to the next generation.
> 3. Any live cell with more than three live neighbours dies, as if by overpopulation.
> 4. Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.

These are our high level requirements. Our goal, if you will. Our
task at this moment is to translate the above into actionable
use cases which we will later translate into JUnit 5 tests. This
doesn't need to be very elaborate (again, just enough to get started).

Beyond the explicit rules listed above, more abstractly the problem is about transforming a grid from a starting state
to the next state. For example, what is the expected outcome if the initial state of the universe if all dead cells? 
According to the rules, nothing. The cells will remain dead into the next generation. As best we can we want our 
scenarios to be expressed in this way, dependent on the observable behavior of the system and not on the specific
implementation.

Given that, I came up with the following scenarios in the form of starting state, expected end state.

```
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
```
I like to keep a list of potential use cases to test, typically as comments in the JUnit 5 test class where I can 
quickly see what scenarios remain. I add tests just above the scenarios, deleting line items as they are promoted to
real tests. This list isn't final. We will be revisiting it frequently, sometimes adding to it and substracting from it
at other times. We may not even complete all the listed items. Often you will find that multiple items are actually
specific examples of the same generic case. This is perfectly acceptable, even expected.

Enough talking, however. Next up we will be writing out first test and officially enter into the TDD workflow!

