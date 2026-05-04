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
  updated: "2026-04-10T08:30-06:00"
---

## Contents

- (a) What it is
- (b) Language(s) & dependencies
- (c) System design
- (d) Example inputs & expected outputs

## (a) What it is

I'm proposing to create a bot "swarm" expansion pack for
[andrew-chang-dewitt/maze-robot][1].

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
will likely utilize some basic libraries for simplifying networking. Ideally,
I will implement some distributed data type abstractions I wish to provide so
that I can learn more about how these types of problems can be solved. This
leaves me with a fairly small dependency graph; however, I haven't fully
researched which rust libraries I will utilize for networking abstraction yet.

## (c) system design

My expansion will eventually need to provide the following features:

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

The existing project's system design can be roughly described as 4 layers:

```no-linenums
1. front-end language  (    high-level language       )
                       (  for programming the robot   )

2. syscall api         (   unified api for exposing   )
                       (     robot capabilities       )

3. robot drivers       ( text-based  ) ( abstractions )
                       ( controller  ) (over ESP robot)

4. host                ( text-based  ) ( ESP embedded )
                       ( mock robot  ) ( cpu/periph.  )
```

Each robot will still need to implement each of these layers, & each of the new
features above will still fit into this design on the individual robot scale.
Additionally, the robots will need to interact as a system as well.To aid in
defining that system, I'll narrow the scope of this project for now to only
implement layers 2, 3, & 4 of the above design as a text-based robot
simulation. this proof of concept will run each robot as a separate program,
each on separate nodes on the FABRIC testbed. this means an individual robot's
system design will look like the follow:

```no-linenums
                          +------ robot -----+
                          |                  |
 { program  }----input----|->[ robot api  ]  |..{ layer 2 }
                          |        | ^       |
 { solution }<----output--|        v |       |
                          |  [ text-based ]  |..{ layer 3 }
                          |  [ controller ]  |
                          |        | ^       |
                          |        v |       |
                          |  [ text-based ]  |..{ layer 4 }
                          |  [ mock robot ]  |
                          |                  |
                          +------------------+
```

a swarm of robots (represented in this implementation as a set of FABRIC nodes
on the same network, each running the text-robot binary) can work together to
solve a maze by the following steps:

1. each bot establishes a swarm connection (see note)
2. then each bot enters the maze (by establishing TCP connection to a single
   node holding the maze data) at the appropriate starting position
3. next, the bots begin executing their maze solution program (in this simple
   implementation, packaged in a binary that consumes the external-facing
   library&mdash;user-provided & executed from memory in later designs)
4. when one bot has found the finish, all bots stop executing

> [!NOTE]
>
> to make configuration as simple as possible, peer-to-peer swarm communication
> is done entirely via UDP broadcast. because the network in FABRIC is entirely
> in my control, issues of congestion are not of any concern here; however,
> they will possibly need to be addressed in actual robot implementations
> designed to run on users' home networks.

which creates the below system a single **maze node (1)** that holds the state
of the maze (location of each bot as well as wall, start, & finish cells) along
with `N` **bot nodes (2)** (where `N > 0`) & an optional **monitor node (3)**.

1. the single **maze node** is queried by participating robot nodes any time a
   robot wants to view/update maze information (e.g. view types of neighboring
   cells or to move to a neighboring cell). these maze state queries are done
   via tcp to ensure state remains consistent.
2. the **bot nodes** share data w/ one another when instructed to by the user
   (or in the example solution program in this implementation) via the
   `swarm-connection` abstraction. this abstraction exposes a means of
   communicating between bots (via rpc style OR message-passing primatives)
   that is transmitted over udp broadcast to a set port that each bot listens
   to.
3. finally, the optional **monitor node** listens on another port for udp
   broadcasts to gather log messages sent by the robot & maze nodes, enabling
   runtime system observability.

```no-linenums
                                                   +====== monitor ======+
                                                  ||                     ||
 +===== bot node(s) =======+     +-·-·-·-·-·-·-·-udp·->[   swarm     ]<-·udp·+
||                         ||    · { to monitor } ||   [ connection  ]   ||  |
|| [---- solution -----]   ||    |                ||         |           ||  ·
|| [---- program  -----]   ||    ·                ||         v           ||  |
||           | ^           ||    |                ||   [   logger    ]   ||  ·
||           v |           ||    ·                ||                     ||  |
|| [---- robot api ----]   ||    |                 +=====================+   ·
||   | ^       | ^         ||<-·-+                                           |
||   | |       v |         ||    |                 +===== maze node =====+   ·
||   | |  [   swarm    ]-·-udp-·-+ { to peers }   ||                     ||  |
||   | |  [ connection ]<-·udp·-·+ { from peers } ||   [ data ][ udp ]·-·udp·+
||   | |                   ||    |                ||     | ^    | ^      ||
||   | |                   ||-·-·+                ||     v |    v |      ||
||   v |                   ||                     ||   [  state api  ]   ||
|| [  dist-bot  ]----------tcp---+                ||         | ^         ||
|| [ controller ]<---------tcp-+ |                ||         v |         ||
||                         ||  | +---------------tcp-->[  dist-maze  ]   ||
 +=========================+   +-----------------tcp---[ comms layer ]   ||
                                                  ||                     ||
                                                   +=====================+
```

## (d) example inputs & expected outputs

A simple, but non-trivial example maze with robot start & finish cells marked
as `S` & `F` & walls as `+`:

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
of maze cells, [see existing example][2]), the set of discovered cells may look
like below, where a dot (`·`) indicates cells traveled by the robot & a space
(` `) indicates cells that were not seen.

```
+++++++ +++++++
+·······+·······F
+++·+·++ ·++++++
 ···+·+  ·······+
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
parallel might generate a solution that looks something like below, where `1`
indicates paths traveled by the first bot, while `2` & `3` indicate paths
traveled by the other two peers.

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
[2]: https://github.com/andrew-chang-dewitt/maze-robot/blob/3baf4e809593e76624a54e459d2a02281dd386b8/src/solution.rs#L17
