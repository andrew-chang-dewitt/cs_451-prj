# bot-swarm

Term project designed & built for Parallel & Distributed Systems course at Illinois Institute of Technology, Spring 2026.

Andrew Chang-DeWitt \
Spring 2026 \
CS 451

---

## about

A extension of andrew-chang-dewitt/maze-robot to enable distributed programming
solutions using a "swarm" of multiple bots running the same program. Currently
a proof of concept, implemented to run on the FABRIC testbed w/ simulated bots
built off of the existing
[`TextRobot`](https://github.com/andrew-chang-dewitt/maze-robot/blob/4f0bf9df1c9f73839c90dfa764062f317a7038c3/src/text_maze/robot_impl.rs#L9)
implementation.

- See [assignment.md](./assignment.md) for given requirements.
- See [docs/design.md](./docs/design.md) for system design.
- Source code is in original project's repo
  [andrew-chang-dewitt/maze-robot](https://github.com/andrew-chang-dewitt/maze-robot/tree/swarm/main)
  with work for this implementation being done on the `swarm/main` branch,
  submoduled here.

## status

Currently a WIP, see TODOs below to get a sense of what's done & what's still
left to be done before this proof-of-concept is functional.

### TODOs

Shared libs:

- [x] Detailed system design
- [ ] Networking abstraction _**[in progress in `swarm/net`]**_

  a layer for simplifying sharing data via message passing (using UDP broadcast
  & TCP connections) to be used by Maze & Robot extensions

Robot features:

- [ ] `DistRobot`, Robot trait method implementations
  - [ ] `DistRobot::get_internal`
- [ ] Add Robot collision functionality
  - [ ] new error to be raised during Robot::go when another Robot is in the way
- [ ] Swarm functionality
  - [ ] `DistRobot::swarm_join() -> Swarm`
  - [ ] `DistRobot::swarm_leave() -> Swarm`

Swarm struct:

- [ ] `Swarm::send<T>(data: T) -> Result<(), ?>`
- [ ] `Swarm::register_listener_for<T, F>(fn: Fn(T, F) -> Result<(), ?>)` where
      F might be SocketAddr or some other way of identifying the message sender
- [ ] `Swarm::start_listening()`
- [ ] `Swarm::stop_listening()`

Maze features:

features necessary to support robot in distributed implementation

- [ ] `DistMazeServer`, a mini (multi-threaded?) server on tcp w/ two endpoints:
  - [ ] `look_dir(Req<Direction>) -> Response<Cell>`
  - [ ] `move_dir(Req<Direction>) -> Response<Result<(), MazeError>>`

- [ ] `DistMazeClient`, for internal use by the `DistRobot` & handles error
      responses so they don't bubble up to the `Robot`
  - [ ] `look_dir(Direction) -> Cell`
  - [ ] `move_dir(Direction) -> Result<(), MazeError>`

Monitor:

a third-party observer process--used for all other nodes to send data too like a stdout buffer

- [ ] Swarm connection to receive messages with
- [ ] logger handler that simply echos messages to stdout

Bonus:

- [ ] A shared data structure built on the message passing features
