---
title: "Introduction"
permalink: /code-smells/introduction/
last_modified_at: 2020-07-03
version: 1.0.0
---
Code smells are features or characteristics of a codebase that, at first glance, just look "off" for lack of a better 
word. The presence of a code smell often indicates a deeper problem worth digging into as they are likely the things 
that leave you confused and make the codebase harder to reason about and maintain. 

The presence of a code smell does not indicate a problem, however, or that the code is bad! The code is likely working 
correctly, and the code smell is unlikely to be a bug. It is simply a signal there may be potential for improvement.

Consider a large class. Most code bases contain one (likely several) large classes, and often these classes play an 
important, sometimes critical, role in that code base working correctly. These classes also tend to be notorious for
their complexity and are frequently a major reason a codebase is difficult to work with or change. If, however, the 
large class has a high internal cohesion, does not prevent the rest of the code base from evolving to meet the needs of 
its users the smell of it being a large class may not be a problem after all.