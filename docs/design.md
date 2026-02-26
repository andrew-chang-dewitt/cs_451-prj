---
title: "`bot-swarm`: extending `maze-robot` to run on a swarm"
description: "Design document created in planning to extend my existing project, `andrew-chang-dewitt/maze-robot`, with the ability to build a swarm of robots & solve mazes using distributed & parallel approach. Details goals & scope, explores a possible system design, & begins to think about test inputs & outputs."
keywords:
  - "project: maze robot"
  - "rust"
  - "distributed systems"
  - "system design"
  - "parallel & distributed"
  - "computer science"
  - "cs 451"
  - "illinois tech"
  - "project planning"
meta:
  byline: Andrew Chang-DeWitt
  published: "2026-02-24T11:30-06:00"
---

## Contents

- (a) What it is
- (b) Language(s) & dependencies
- (c) System design
- (d) Example inputs & expected outputs

## (a) What it is

I'm proposing to create a bot "swarm" expansion pack for [andrew-chang-dewitt/maze-robot][1].

The original project is a teaching toy for young children to learn programming
skills without the use of screens. Instead of writing code in text, they build
programs using physical puzzle pieces (similar to Scratch). When they're ready
to try out their solution, they have the robot scan the puzzle pieces by
driving over them, after which it compiles the program to RISC-V machine code,
then executes the program to explore a maze & find the designated finish point.

As part of creating that project, I've written a text-based test environment to
have a simulated robot explore mazes as well. My intention for this course is
to expand on this environment by creating a set of new robot commands that can
be used to link multiple robots together, then create a set of abstractions
that the user can build on to easily create distributed programs that
coordinate a "swarm" of robots to solve the maze.

<!--
- goals
  - user outcomes
  - personal outcomes
  - stretch goals
    - scaling across massive mazes (e.g. loading zones in mmo games)
    - graduated introduction to parallelism & distributed programming
      (by providing very high-level abstractions, plus exposing more
      limited versions as mid- & low-level abstractions & asking user
      to implement their own abstractions as needed)

- maybe user stories showing motivating example meeting goals?

  ...
-->

## (b) language(s) & dependencies

Because the already existing code base is built in Rust, I intend to continue
implementing these new features in Rust as well. As far as dependencies go, I
will likely utilize some basic libraries for simplifying network
communications. Ideally, I will implement the distributed data type abstraction
I wish to provide so that I can learn more about how these types of problems
can be solved. This leaves me with a fairly small dependency graph; however, I
haven't fully researched which rust libraries I will utilize for networking
abstraction yet.

## (c) system design

My expansion will need to provide the following features:

1. A new "syscall" exposing a new capability to connect two robots over a
   network, implemented in the robot controller library.

   Ideally both long-lasting socket-type connections & more ephemeral
   message-passing connection will be possible.

2. Logic for handling physical interactions between robots, including how to
   differentiate between a maze wall & another robot blocking a path

3. Some low-level distributed message-passing abstractions, such as condition
   variables, signals, &/or simple pub-sub/channel style messaging tools

4. Some low-level distributed data structures (such as shared "memory" via
   "mutexes" & "refcells") & some higher-level ones (such as distributed a
   queue/stack)

5. Front-end language implementations of these features, allowing users of the
   accompanying simple learning language to build programs using bot swarms

The existing project's system design can be roughly described as 3 layers:

```
front-end language  (    high-level language       )
                    (  for programming the robot   )

syscall api         (   unified api for exposing   )
                    (     robot capabilities       )

robot drivers       ( text-based  ) ( abstractions )
                    ( controller  ) (over ESP robot)

host                ( text-based  ) ( ESP embedded )
                    ( mock robot  ) ( cpu/periph.  )
```

Each robot will still need to implement each of these layers, & each of the new
features above will still fit into this design on the individual robot scale.
Additionally, the robots will need to interact as a system as well. To aid in
defining that system, I'll narrow the scope of this project for now to be just
a text-based robot simulation, which will run each robot as a separate program,
each on separate nodes on FABRIC. For a swarm of 5 robots initialized by having
one robot (main) establish connections to each new robot, introduce the new
robot to the swarm (allowing all to communicate directly in a mesh if desired),
then main scans/compiles the program before beginning execution & coordinating
robots as instructed by the user proram.

## (d) example inputs & expected outputs

A simple, but non-trivial example maze with robot start & finish cells marked
as "S" & "F" & walls as "+":

```
+++++++++++++++++
+       +       F
+++ + +++ +++++++
S   + +         +
+ + +++++++++++ +
+++           + +
+ + ++ ++ +++ + +
+   ++    +   + +
+++++++++++ +++ +
+           + + +
+ +++++++++++   +
+             + +
+++++++++++++++++
```

If solved with a single-bot algorithm (implemented as a DFS search of a graph
of maze cells, [see existing
example](https://github.com/andrew-chang-dewitt/maze-robot/blob/3baf4e809593e76624a54e459d2a02281dd386b8/src/solution.rs#L17)),
the set of discovered cells may look like below, where "·" indicates cells
traveled by the robot & " " indicates cells that were not seen.

```
 +++++++ +++++++
+·······+·······F
+++·+·++ ·++++++
S···+·+  ·······+
   ·+++++++++++·
   ···········+·
           ++·+·
           ···+·
 ++++++++++·++ ·
 ···········+·+·
 ·+++++++++++···
 ·············

```

A similar, DFS-based, approach for a swarm of 3 bots working on the maze in
parallel might generate a solution that looks something like below, where "1"
indicates paths traveled by the first bot (main), while "2" & "3" indicate
paths traveled by the other two peers.

```
+++++++++++++++++
+1111111+3333333F
+++1+1+++3+++++++
S111+1+223333333+
+3+2+++++++++++3+
+++22222222222+3+
+3+3++3++1+++2+3+
+333++3331+222+3+
+++++++++++2+++3+
+22222222222+2+3+
+2+++++++++++211+
+2222222222222+1+
+++++++++++++++++
```

[1]: https://github.com/andrew-chang-dewitt/maze-robot
