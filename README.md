# puzzle_design
A game engine for generic puzzle design and problem solving

### What is puzzle design and problem solving?

Puzzle design and problem solving is commonly used to make games more interesting. Some times, problem solving is part of the gameplay. Other times, problem solving is part of the content in a game.

For example, a popular algorithm for generating procedural content is [Wave Function Collapse](https://github.com/mxgmn/WaveFunctionCollapse). This algorithm is based on constraint solving and uses backtracking with entropy. To implement a similar algorithm here, you can use `puzzle_design::quickbacktrack::EntropySolver`.

This library is for general problem solving.
The algorithms can be used to solve puzzles that look different from each other and might use custom Rust data structures.

### Design

This library reexports the following:

- [avalog](https://github.com/advancedresearch/avalog) for Avatar Logic (a graph-like version of Prolog)
- [discrete](https://github.com/advancedresearch/discrete) for discrete spaces
- [linear_solver](https://github.com/advancedresearch/linear_solver) for automated generic linear solver
- [monotonic_solver](https://github.com/advancedresearch/monotonic_solver) for automated generic monotonic solver
- [piston_mix_economy](https://github.com/advancedresearch/mix_economy) for economic constraint solving without backtracking
- [quickbacktrack](https://github.com/advancedresearch/quickbacktrack) for constraint solving with backtracking

## Introduction

A constraint is a rule that limits the space of solutions of a puzzle.

For example, assume there is this simple puzzle: Alice lives on the side of her street with odd umbers. Her house number is larger than `123` and less than `321`.

In this puzzle, there are 3 constraints:

1. Alice's house number is odd.
2. Alice's house number is larger than `123`.
3. Alice's house number is less than `321`.

You can also think about these 3 constraints as 3 facts.

When we solve problems using a solver, we need to program the solver
with inference rules such that it makes progress.
E.g. in Avalog, you can specify rules that it uses to make inference (to think).

- A monotonic solver can only add new facts, not erase them
- A linear solver can add and remove facts

For example, Avalog is a monotonic solver, so it can not remove facts.

A linear solver can e.g. take 2) and 3) and replace them with an open range `(123, 321)`. Next, it can replace the open range with a closed range `[122, 320]`. Finally, it can use 1) to replace the closed range with `[125, 319]`.

Each of these steps require inference rules. Without inference rules, the solver gets stucik.

Notice that this is not enough information to tell in which house where Alice lives. There is more than one solution:

- A puzzle that has more than one solution is under-constrained.
- A puzzle that has no solution is over-constrained.

In game design, you can cheat. For example, you can make the player meet Alice at the 3rd house they try, regardless of which houses they visit and in which order. From that point onward, you have to keep track of where the player met Alice, so that the story appears coherent to the player.

### When linear solvers run in circles

Linear solvers are very powerful, because they can transform the data into a more useful form. They stop when there is no work left, or when they detect that they are running in circles.

For example, imagine that the linear solver first reduce the puzzle of Alice's house to 2 facts, but later expanded to 3 facts:

1. Alice's house number is odd.
2. Alice's house number is greater or equal to `125`.
3. Alice's house number is less or equal to than `319`.

The linear solver has improved the information a little,
but when it keeps going, it repeats the same 3 facts/constraints.

By keeping track of previous states, the linear solver can detect that it
has seen this state before. Assuming determinism, it runs an extra round and finds the state with a minimum number of facts:

1. Alice's house number is odd.
2. Alice's house number is in the closed range `[125, 319]`.

There, the solver stops, so you can use this information for further problem solving.

### Why use a monotonic solver instead of linear

Monotonic solvers terminate when there are no new facts to be added.
This means, monotonic solvers do not need cycle detection.

Monotonic solvers also terminate when they reach their goal.
This makes them capable of handling infinite number of new facts.

### Backtracking

In some puzzles, we are only interested in finding some solution, if it exists. Instead of thinking about the problem at higher level, we might just try something and see what happens. When it does not work, we go back to an earlier state and try something else. This is the basic principle of backtracking.

### Economic problem solving without backtracking

In economics, backtracking might not always be possible.

This is because economics is about problems that require multiple agents to solve them.
When agent A tries to backtrack to an earlier state, the environment might have changed because A's choice depends on another agent B's state.
For A to truly backtrack, it might need to undo both its own actions and B's actions. Now, agent B might not agree with this change, because it predicts that it will end up in a worse economic situation.

In addition to the coordinate problem of backtracking, some actions might change the environment forever, making it practically impossible to backtrack in general.

To enable voluntary cooperation between agents in an economy,
the agents might agree upon a shared contract where they optimize the inequality gradient using a target Gini coefficient.
This might reduce the worst case scenario and boost the economy's ability to solve constraint problems, without backtracking.
