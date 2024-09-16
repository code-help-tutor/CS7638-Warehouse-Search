
# CS 7638: Artificial Intelligence for Robotics
# Warehouse Project - Summer 2024 - Deadline: Wednesday July 10th, Midnight AOE

## Introduction
This PDF serves as a problem specification document and thus the information in this document is not referring to any details regarding any specific algorithm to solve the problem. Its’ purpose is to define the problem along with the relevant details needed to solve it. It acts as a summary of the rules that are implemented in the testing suite code provided to you (which you can reference when you need further clarification). An interactive GUI mode (TEST_MODE) is also available for you to use (which you can use to validate your understanding of the specifications). Lastly, the example calculations and images in this PDF are also a good resource to use when looking for clarifications.

In this project, you will implement search algorithms to guide a robot through a warehouse to pick up and drop off boxes to a designated drop zone area.

The template code provides 3 classes, one for each part of the project: `DeliveryPlanner_Part[A, B, C]`
- You may share code between part A, B, and C
- Your submission will consist of a single file: `warehouse.py`

## Grading
- The weighting for each part is:
    - Part `A = 40%`
    - Part `B = 40%`
    - Part `C = 20%`
- Within each part there are 10 test cases, each test case is equally weighted.
- No assumptions should be made about any similarities between local and GS test cases. All test cases will adhere to the rules laid out in this PDF. Your solution doesn’t need to handle test cases that fall outside of these rules.

## Part A (40%)
Your task is to pick up and deliver all boxes listed in the todo list. You will do this by providing a list of motions that the testing suite will execute in order to complete all the deliveries. Your algorithm should determine the optimal path to take while completing this task.

`DeliveryPlanner_PartA`'s constructor must take five arguments: `self`, `warehouse_viewer`, `dropzone_location`, `todo`, and `box_locations`. It must also have a method called `plan_delivery` that takes 2 arguments: `self` and `debug` (set to `False` by default).

`plan_delivery` will only be invoked once for each test case. The testing suite will not execute code while your function is creating a solution to deliver ALL boxes in the todo list. This means that your warehouse map will not automagically update during your function invocation, you will need to perform all updates to your warehouse map representation as necessary for processing your solution.

### Part A Input Specifications
`warehouse_viewer` will be a custom object used by the testing suite. For all intents and purposes, you can think of it as a list of `m` lists, each inner list containing `n` characters, corresponding to the layout of the warehouse. The warehouse is an `m x n` grid. `warehouse[i][j]` corresponds to the spot in the `ith` row and `jth` column of the warehouse, where the 0th row is the northern end of the warehouse and the 0th column is the western end.

NOTE: In part A, your code will not know (nor should it depend on) the size of the warehouse (`n` and `m`). You are only allowed to use the `warehouse_viewer` object (WarehouseAccess) in the following 2 ways, if we find that your code is using any other methods/approaches to use or manipulate the warehouse object, you will receive a 0 for part A (this may be done manually after the project is due):
1. You may access a particular cell in the warehouse using `warehouse[i][j]`
2. You may overwrite the contents of a cell using `warehouse[i][j] = '<some symbol>'` (note that this is only for your convenience if needed)

The goal in part A is to not only find an optimal solution, but to do so in an efficient manner. Efficiency here means that your algorithm should access (view) as few of the warehouse cells as possible. Accessing a particular cell more than once will NOT hurt you as we will only tally the unique cells that your algorithm accesses each test case. It is your responsibility to make sure you are not using any other way to glean information from the warehouse other than the 2 methods above. There are some precautions in place to help notify you when you are improperly using the warehouse object, however, these are not exhaustive. A few things to ask yourself (if you answer yes to any of these, you are likely approaching the problem incorrectly and likely going to receive a 0 for this part):
1. Are you attempting to determine the size of the warehouse?
2. Are you using the `len()` function on the warehouse or its’ rows/cols?
3. Are you attempting to access an entire row of the warehouse (rather than a particular cell)?
4. Are you accessing a private attribute (as indicated by a leading underscore) of the warehouse?
5. Are you iterating through the warehouse in any way?
6. Are you somehow gaining access to a warehouse cell’s contents without it being counted towards the viewed_cell_count?
7. Are you attempting to copy the warehouse object/data in any way?
8. Are you printing the warehouse object?

Note that the box locations and the dropzone are counted as viewed cells because they are given to you as input.

The characters in each string will be one of the following:
- `. (period)`: traversable space. The robot may enter from any adjacent space.
- `# (hash)`: a wall. The robot cannot enter (or exist at) this space. [The following applies ONLY to part A: All warehouse cases will be surrounded by walls].
- `@ (dropzone)`: the starting point for the robot and the space where all boxes must be delivered. The dropzone may be traversed like a `. (period)`.
- `[0 - 9a - zA - Z] (any alphanumeric character)`: a box. At most one of each alphanumeric character will be present in the warehouse (meaning there will be at most 62 boxes). A box may not be traversed, but if the robot is adjacent to the box, the robot can pick up the box. Once the box has been lifted, the space that the lifted box previously occupied now functions as a `. (period)`.

For example,
```
warehouse = ['#####'
'#1#2#'
'#.#.#'
'#####']
@#
```
is a 5x5 warehouse.
- The dropzone is at the warehouse cell in row 3, column 3.
- Box 1 is located in the warehouse cell in row 1, column 1.
- Box 2 is located in the warehouse cell in row 1, column 3.
- There are walls within the warehouse at cells (row 1, column 2) and (row 2, column 2) and around the warehouse.
- The remaining five warehouse cells (which includes the dropzone) are traversable spaces.

The argument `todo` is a list of alphanumeric characters giving the order in which the boxes must be delivered to the dropzone. For example, if `todo = ['1','2']` is given with the above example warehouse, then the robot must first deliver box 1 to the dropzone, and then the robot must deliver box 2 to the dropzone.

### Part A Rules & Costs for Motions
- The robot may move in 8 directions (`N`, `E`, `S`, `W`, `NE`, `NW`, `SE`, `SW`)
- The robot may not move outside the warehouse. The warehouse does not “wrap” around (it is not cyclic).
- Two spaces are considered adjacent if they share an edge or a corner.
- The robot may pick up a box that is in an adjacent square.
- The robot may put a box down in an adjacent square, so long as the adjacent square is empty (`. or @`).
- While holding a box, the robot may not pick up another box.
- There are 4 kinds of motions that the robot can take (below are the costs associated with each type):
    - `[cost]: type`
    - `[2]`: horizontal or vertical movement
    - `[3]`: diagonal movement
    - `[4]`: pick up box (regardless the direction)
    - `[2]`: put down box (regardless the direction)
- If a box is placed on the `@` space, it is considered delivered and is removed from the warehouse, thus the `@` space is still traversable after dropping a box on it.
The warehouse will be arranged so that it is always possible for the robot to move to the next box on the todo list without having to rearrange any other boxes.
- The robot will end up in the same location when an illegal motion is performed.
- An illegal motion will incur a penalty cost of 100 in addition to the motion cost.
- Illegal motions include:
    - attempting to move to a nonadjacent, nonexistent, or occupied space
    - attempting to pick up a nonadjacent or nonexistent box
    - attempting to pick up a box while already holding one (attempting to put down a box while not holding one)
    - attempting to put down a box on a nonadjacent, nonexistent, or occupied space (this means the robot may not drop a box on the drop zone while the robot is occupying the drop zone)

### Part A Method Return Specifications
`plan_delivery` should return a list of moves that minimizes the total cost of completing the task. Each move should be a string formatted as follows:
- `'move {d}', where '{d}' is replaced by the direction the robot should move: "n", "e", "s", "w", "ne", "se", "nw", "sw"`
- `'lift {x}', where '{x}' is replaced by the alphanumeric character of the box being picked up`
- `'down {d}', where '{d}' is replaced by the direction the robot will put the box down`

For example, for the values of `warehouse` and `todo` given previously (reproduced below):
```
warehouse = ['#####',
'#1#2#', '#.#.#',
'#..@#',
'#####']
todo = ['1','2']
```
`plan_delivery` might return the following:
```
['move w',
'lift 1', 'move nw', 'move se',
'down e',
'lift 2', 'move ne', 'down s']
```

### Part A Scoring
The testing suite will execute your plan and calculate the total cost: `student_cost`. The score for each test case will be calculated by: `benchmark_cost / student_cost`. The benchmark will be greater than or equal to the absolute minimum cost. You will receive a 0 in the following situations:
- your code views more warehouse cells than specified in the test case: `viewed_cell_count_threshold`
- your code takes longer than the prescribed time limit
- your method returns the wrong output format
- the boxes are not delivered in the correct order

## Part B (40%)
In this part there are 3 main differences from part A:
1. there will be only a single box for your robot to deliver
2. the warehouse has an “uneven” floor which imposes an additional cost (range: 0 ~ 95 inclusive)
3. the robot starting location is not provided

`DeliveryPlanner_PartB`'s constructor must have four arguments: `self`, `warehouse`, `warehouse_cost`, and `todo`.

### Part B Input Specifications
Same as part A with the following exceptions:
- The only box in the warehouse will be: `1` (the single box to be delivered).
- The warehouse will NOT necessarily be surrounded by walls.
- There are NO restrictions on your code accessing the warehouse object in part B and C. You may view as many cells as you wish.

Note: Test cases in the test suite will only contain the characters listed in part A’s input specifications section. There is a helper function `(_set_initial_state_from)` that parses this initial input into an internal warehouse state. This is the same internal state representation that is used by the testing suite. You are NOT required to use this helper function and may change it as you see fit, it is just provided for convenience. Note that the testing suite uses an asterisk (`*`) to denote the current location of the robot as it executes your plan. This asterisk is only used for internal purposes in the testing suite so you will not see it present in the test cases. It is also used to denote the robot’s location in some examples in this document.

For example:
```
warehouse = ['1..',
'....', '.##.']
```
The argument `warehouse_cost` is a list of lists such that indices `i,j` refer to the floor cost at the row `i` and column `j` in the warehouse. For the case above, the corresponding `warehouse_cost` could be:
```
warehouse_cost = [[ 0, 5, 2],
[10, w, 2], [ 2, 10, 2]]
```
where `w` represents a wall. Note that the value of `w` has no consequence since the robot can’t occupy a space containing a wall.

The argument `todo` is limited to a single box as follows: `todo = ['1']`

There is no input for initial robot location because the robot may “wake up” at any point in the warehouse and must be handed a “policy” so that no matter where it is, it can retrieve the box. Further, because it may lift the box from different squares depending on its starting location, it requires another “policy” to deliver the box to the dropzone.
- Note: You must update your internal warehouse state in your code as this is not done for you.

### Part B Rules for Motions
Same as part A.

### Part B Costs for Actions
The total cost for an action consists of a summation of 2 parts:
- motion cost (same as part A)
- floor cost
    - movements: value of the destination cell the robot is moving into
    - lift: value of the cell the box is located in prior to lifting
    - down: value of the cell the box is being placed into

This means although you may incur less motion cost to move straight to a target location, the additional floor cost along the way may be such that taking a roundabout way will result in an overall lower cost.

For example the lowest cost route to box 1 is not `[‘move e’, ‘move e’]`:
```
warehouse = ['*..1',
'....', '.##.']
warehouse_cost = [[ 1, 95, 50, 1], [ 1, 1, 1, 1], [ 1, w, w, 1],]
```
Two example calculations for the total cost of an action using the example grid above are:
- If the robot enters (0,1) from (0,0) then the total action cost will be: `total action cost = motion cost (horizontal movement) + floor cost (destination) = 2 + 95 = 97`
- If the robot enters (0,1) from (1,0) then the total action cost will be: `total action cost = motion cost (diagonal movement) + floor cost (destination) = 3 + 95 = 98`

Note that the floor cost to move into cell (0,1) is 95 regardless of the direction the robot is entering from. Three example calculations for the total action cost of illegal motions (i.e. attempting to move into (or put down a box at) an occupied space or outside the warehouse) are:
- If the robot attempts to move east from (2,0) then the total action cost will be: `total action cost = motion cost (horizontal movement) + illegal motion penalty cost = 2 + 100 = 102`
- If the robot attempts to move southeast from (2,0) then the total action cost will be: `total action cost = motion cost (diagonal movement) + illegal motion penalty cost = 3 + 100 = 103`
- If the robot attempts to put down a box to the southeast from (2,0) then the total action cost will be: `total action cost = motion cost (put doun boz) + illegal motion penalty cost = 2 + 100 = 102`

Note that the motion costs are still included in the case of illegal motions even though they weren’t successful (the robot still exerted the energy). Floor costs are only incurred when a motion is legally carried out. Floor costs (of the box location) are incurred when legally lifting, and floor costs (of the drop location) are incurred when putting down boxes.

### Part B Method Return Specifications
`plan_delivery` should return two policies, each as a list of lists of strings indicating the motion to take at each square on the grid. The format of the commands is the same as in part A. The special command `‘-1’` should be placed at any square for which there is no valid command, such as a wall.

For example, for the values of `warehouse` and `todo` given previously (reproduced below):
```
warehouse = ['1..',
'....', '.##.']
```
`plan_delivery` might return the following two policies:
To Box Policy:
```
[['B', 'lift 1', 'move w' ]
['lift 1', '-1', 'move nw'] ['move n', 'move nw', 'move n' ]]
```
Deliver Box Policy:
```
[['move e', 'move se', 'move s']
['move ne', '-1', 'down s'] ['move e', 'down e', 'move n']]
```
where: `‘B’` indicates the box location.

For the “Deliver Box Policy”, the dropzone includes a motion in the event the robot starts on, lifts an adjacent box, and then must move off the dropzone to deliver it.

# CS7638 Warehouse Search

# CS Tutor | 计算机编程辅导 | Code Help | Programming Help

# WeChat: cstutorcs

# Email: tutorcs@163.com

# QQ: 749389476

# 非中介, 直接联系程序员本人
