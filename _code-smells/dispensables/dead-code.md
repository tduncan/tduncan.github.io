---
title: "Dead Code"
permalink: /code-smells/dispensables/dead-code/
last_modified_at: 2020-07-17
---
Dead code is code that is no longer referenced by other parts of the production codebase, often because it has been made
obsolete.

At first glance Dead Code may seem harmless, but it most definitely is not. At minimum, it is purely unnecessary code
bloat, but at worst Dead Code has the potential to
[ruin a company](https://en.wikipedia.org/wiki/Knight_Capital_Group#2012_stock_trading_disruption).

Additionally, Dead Code has a habit of obscuring options for potential improvement as well as acting as additional 
friction when considering changes in parts of the codebase that may require the Dead Code to be updated.

#### Identification
A good IDE will help immensely for identifying Dead Code. Most IDEs will automatically flag a block of Dead Code with a
warning indicator, often accompanied by a much lighter font color to further draw attention to the problem.

Do be careful with code that your IDE may have the usage context available for. The external APIs of a library or
framework glue code are two examples of code that may be flagged as Dead Code, but has real usages the IDE is unable
understand.

#### Remedy:
Fortunately Dead Code is easy to fix. Delete it.

#### Variations:
Unused import
Unused constant
Unused field
Unused method parameter
Unreachable code block within a method (think code within if/else if/else)
Unused method
Unused class
Unused interface