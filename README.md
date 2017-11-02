Homework 3: Candy Rush
This homework is due on Thursday, October 12, at 11:59:59 PM (Eastern daylight time). You must use submit to turn in your homework like so: submit cs421_jtang hw3 hw3.c

Your program must be named hw3.c, and it will be compiled on Ubuntu 16.04 as follows:

  gcc ‐‐std=c99 ‐Wall ‐O2 ‐o hw3 hw3.c ‐pthread ‐lm
(Note the above is dash, dash, "std=c99", and the other flags likewise are preceded by dashes.) There must not be any compilation warnings in your submission; warnings will result in grading penalties. In addition, your code file must be properly indented and have a file header comment, as described on the coding conventions page.
With Halloween approaching, you have been tasked to optimize the best route for groups of children trick-or-treating. Your program will simulate a neighborhood of generous candy givers with the households gradually replenishing their candy supplies. You will also simulate groups walking the neighborhood streets obtaining that candy. Your goal is to maximize the amount of candy received during the simulation time.

Part 1: Starting Up
Your C program accepts two single command-line arguments: the name of a data file and the amount of time to simulate. Call this simulation time T. Your program will parse the data file to get the locations of houses, and the number and size of trick-or-treating groups. The file then contains how fast those houses replenish their candy. If the user does not provide two arguments, or if the data file is unreadable, then display an error message and quit. Otherwise, parse the file as described in Part 3 below.

Your program will spawn these threads:

One thread for each of the trick-or-treating children groups.
One thread for the neighborhood in general; it will be refilling houses' candies.
The general flow for your program will be:

Parse the initial data file contents.
Initialize all global data values, including all necessary locks.
Spawn threads for each group.
Spawn the neighborhood thread.
Meanwhile, in the main thread, once per second, display the status of the entire simulation.
After time T, end the simulation. Each children group thread completes its current travel and then terminates itself. The neighborhood thread group also terminates itself.
The main thread waits for all threads to terminate. It then displays one final status of the simulation.
Part 2: Data File Format
The data file format is as follows:

All lines are newline separated; no line will exceed 80 characters in length. Consider using fgets() to read each line from the file.
The first line will contain a positive integer. This is the number of children groups that are trick-or-treating. Call this number G.
The next 10 lines give the coordinates of houses and their initial candy count, as space-separated integers. See Part 3.
The next G lines give the initial positions of the children and how many are in each group. See Part 3.
All remaining lines in the data file describe how houses restock their candy. Each line consists of two fields that are space-separated: house number, and how much candy to add. All fields are unsigned integers. See Part 4.
Within your Ubuntu VM, use the wget command to fetch the data file http://www.csee.umbc.edu/~jtang/cs421.f17/homework/trick1.data. The grader will use a different data file with different filename during grading. You may not make any assumptions about the number of children groups, simulation time, or how much candy is at each house. It is possible the neighborhood thread will not have processed all remaining lines in the data file before the simulation ends.

Part 3: Houses and Children Groups
There are 10 houses in this neighborhood. Each houses' location is described by the coordinates (X, Y), where X and Y are unsigned integers no greater than nine. In this city, the position (0, 0) is the southwest corner, (9, 0) is the southeast corner, and (9, 9) is the northeast corner.

The first line in the data file (i.e., second line of the file) corresponds with house 0, the next line is house 1, and so forth up to house number 9. On each line, the first two numbers give that house's coordinates. The third number is how much candy is initially at that house. (Some houses may begin with no candy at all.)

After parsing the first 11 files of the data file, start spawing children group threads. The next G lines contain two fields: starting house number and the number of children in that group. Create a thread and assign it a unique identifier. For each group, initialize a counter to zero; this counter tracks the total amount of candy that group has received.

Each children group travels to various houses to trick-or-treat. The travel time to each house is equal to the Manhattan distance multiplied by 250 milliseconds. Simulate this travel time by sleeping the thread. For example, travelling from (4, 2) to (1, 0) would require 1250 milliseconds. When the group arrives at the house, increment that group's candy count by the size of the group and decrement the house's current candy amount. If there are more children in the group than candy available, then the group takes all remaining candy; the house's candy amount is set to zero, at least until it is replenished (see Part 4).

Note that a group may go to the same house multiple times, but may not trick-or-treat consecutively at the same house. It must go to a different house before it can return. Groups may not trick-or-treat at their starting house throughout the simulation.

Part 4: Neighborhood Thread
The remaining lines of the data file are to be processed by the neighborhood thread. Each line contains a house number and how much to add to that house's candy supply. Be aware that some lines will have 0 for the amount to add.

When the simulation begins, have the neighborhood thread sleep for 250 milliseconds. Then, if the simulation has not ended, read the next line. Add the amount (potentially 0) to the house's supply. Repeat by sleeping again and then processing the next line.

If the neighborhood thread runs of lines to process, then the thread terminates itself, even if the simulation has not ended.

It is possible for a children group to arrive at a house at the exact same time that the neighborhood thread is refilling that house's candy supply. You may implement any strategy to resolve this race condition, as long as children do not take candy that the house does not already have.

Part 5: Main Thread
While your children group threads are busy walking around the virtual neighborhood and neighborhood thread is replenishing houses, the main thread continues processing. Once per second, up to T seconds, display these values:

Destination of each children group.
Amount of candy each children group has already collected.
Amount of candy remaining at each house.
Sum of all candy received by the children.
After T seconds elapsed, signal to every children group thread to terminate. (How you signal is up to you.) Those threads are permitted to finish travelling to the house and trick-or-treat one more time. Also signal the neighborhood thread; that thread ends itself as soon as possible.

After all threads have terminated, display the simulation status once more.

Part 6: Required Documentation
In your code, add a comment block that answers the following:

While the main thread is displaying the current state of all children groups, there is [at least one] potential race condition. Describe a scenario that could trigger that race condition.
How could your program prevent that race condition? No code necessary, just describe what you would do.
Sample Output
Here is a sample output from running the program against the above sample data file for four seconds. This program has added extra debugging output, indicated in italics, to display where each group is traveling and when the neighborhood thread is adding candy.

$ ./hw3 trick1.data 4
Group 0: from house 0 to 5 (travel time = 750 ms)
Group 2: from house 1 to 0 (travel time = 2250 ms)
Group 1: from house 8 to 3 (travel time = 1250 ms)
After 0 seconds:
  Group statuses:
    0:   size 12, going to 5, collected 0
    1:   size 7, going to 3, collected 0
    2:   size 2, going to 0, collected 0
  House statuses:
    0 @ (2, 6): 2 available
    1 @ (8, 9): 0 available
    2 @ (3, 3): 2 available
    3 @ (3, 4): 8 available
    4 @ (0, 3): 9 available
    5 @ (2, 3): 44 available
    6 @ (1, 4): 5 available
    7 @ (5, 1): 43 available
    8 @ (1, 7): 0 available
    9 @ (0, 8): 3 available
  Total candy: 0
Neighborhood: added 2 to house 7
Neighborhood: added 6 to house 3
Group 0: from house 5 to 3 (travel time = 500 ms)
Neighborhood: added 1 to house 9
After 1 seconds:
  Group statuses:
    0:   size 12, going to 3, collected 12
    1:   size 7, going to 3, collected 0
    2:   size 2, going to 0, collected 0
  House statuses:
    0 @ (2, 6): 2 available
    1 @ (8, 9): 0 available
    2 @ (3, 3): 2 available
    3 @ (3, 4): 14 available
    4 @ (0, 3): 9 available
    5 @ (2, 3): 32 available
    6 @ (1, 4): 5 available
    7 @ (5, 1): 45 available
    8 @ (1, 7): 0 available
    9 @ (0, 8): 4 available
  Total candy: 12
Neighborhood: added 4 to house 2
Group 1: from house 3 to 4 (travel time = 1000 ms)
Group 0: from house 3 to 5 (travel time = 500 ms)
Neighborhood: added 9 to house 5
Neighborhood: added 9 to house 6
Group 0: from house 5 to 6 (travel time = 500 ms)
Neighborhood: added 7 to house 0
After 2 seconds:
  Group statuses:
    0:   size 12, going to 6, collected 31
    1:   size 7, going to 4, collected 7
    2:   size 2, going to 0, collected 0
  House statuses:
    0 @ (2, 6): 9 available
    1 @ (8, 9): 0 available
    2 @ (3, 3): 6 available
    3 @ (3, 4): 0 available
    4 @ (0, 3): 9 available
    5 @ (2, 3): 29 available
    6 @ (1, 4): 14 available
    7 @ (5, 1): 45 available
    8 @ (1, 7): 0 available
    9 @ (0, 8): 4 available
  Total candy: 38
Neighborhood: added 5 to house 9
Group 2: from house 0 to 2 (travel time = 1000 ms)
Group 1: from house 4 to 0 (travel time = 1250 ms)
Group 0: from house 6 to 5 (travel time = 500 ms)
Neighborhood: added 7 to house 3
Neighborhood: added 0 to house 1
Group 0: from house 5 to 7 (travel time = 1250 ms)
Neighborhood: added 0 to house 8
After 3 seconds:
  Group statuses:
    0:   size 12, going to 7, collected 55
    1:   size 7, going to 0, collected 14
    2:   size 2, going to 2, collected 2
  House statuses:
    0 @ (2, 6): 7 available
    1 @ (8, 9): 0 available
    2 @ (3, 3): 6 available
    3 @ (3, 4): 7 available
    4 @ (0, 3): 2 available
    5 @ (2, 3): 17 available
    6 @ (1, 4): 2 available
    7 @ (5, 1): 45 available
    8 @ (1, 7): 0 available
    9 @ (0, 8): 9 available
  Total candy: 71
Neighborhood: added 4 to house 5
Group 2: from house 2 to 4 (travel time = 750 ms)
Neighborhood: added 6 to house 0
Group 1: from house 0 to 3 (travel time = 750 ms)
Neighborhood: added 4 to house 9
Neighborhood: added 0 to house 7
Group 2: from house 4 to 6 (travel time = 500 ms)
After 4 seconds:
  Group statuses:
    0:   size 12, going to 7, collected 67
    1:   size 7, going to 3, collected 28
    2:   size 2, going to 6, collected 8
  House statuses:
    0 @ (2, 6): 6 available
    1 @ (8, 9): 0 available
    2 @ (3, 3): 4 available
    3 @ (3, 4): 0 available
    4 @ (0, 3): 0 available
    5 @ (2, 3): 21 available
    6 @ (1, 4): 0 available
    7 @ (5, 1): 33 available
    8 @ (1, 7): 0 available
    9 @ (0, 8): 13 available
  Total candy: 103
Other Hints and Notes
Ask plenty of questions on the Blackboard discussion board.
At the top of your submission, list any help you received as well as web pages you consulted. Please do not use any URL shorteners, such as goo.gl or tinyurl.
Assume the data file will correctly parse.
Each house requires a lock, for the situation where multiple children are arriving simultaneously and/or the neighborhood thread is adding candy to that house.
Use the usleep() function to make a thread sleep. Note that this function sleeps in increments of microseconds, so make sure you multiply the sleep time by 1000, to convert from milliseconds to microseconds.
Extra Credit
In the sample output above, children groups head to the nearest house that has at least enough candy to for the group's size. In this simplistic algorithm, the groups are not coordinating to maximize their candy.

As a friendly competition, and for fame and glory, you may optimize your algorithm. You can earn an additional 10% on this assignment if your children groups consistently get more candy than the instructor's reference implementation for a different (and larger) data file.

All extra credit submissions will be tested on the same virtual machine. This virtual machine will have more than one CPU allocated to it. Implementations may not cheat, including but not limited to:

Spawning more than G threads.
Not sleeping enough when traveling between houses.
Allowing a house's candy supply to ever drop below zero.
Trick-or-treating at the same house consecutively.
Trick-or-treating at the group's starting house.
The instructor is the final arbitrator of what is considered cheating or not.
If you choose to perform this extra credit, put a comment at the top of your file, alerting the grader.
