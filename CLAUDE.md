# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

`bot-swarm` — CS 451 (Parallel & Distributed Systems, IIT Spring 2026) term project extending [`andrew-chang-dewitt/maze-robot`](https://github.com/andrew-chang-dewitt/maze-robot) with distributed swarm capabilities. Source lives in the `maze-robot/` git submodule, tracking the `swarm/main` branch of the upstream repo.

## Commands

All commands run from `maze-robot/`:

```sh
cargo build
cargo test                                           # unit tests across all modules
cargo test --example text-dfs                        # example-embedded integration tests
cargo run --example text-dfs -- examples/test-maze.txt
cargo run --example text-dfs --features verbose -- examples/test-maze.txt
bacon run-ex-text-dfs-verbose                        # watch mode (bacon.toml)
```

The `verbose` feature enables extra logging in the DFS search.

## Architecture

```
traits/         Core abstractions
  Maze          look_dir(dir) + move_dir(dir) — environment the robot lives in
  Robot         peek(dir) + peek_all() + go(dir) — delegates to RobotInternal
  RobotInternal holds RefCell<Box<dyn Maze>>; interior mutability lets go() take &self
  MazeError     CreationError | MoveError

text_maze/      Local simulation (complete)
  TextMaze      implements Maze; parses text grids (S=start, F=finish, +=wall, space=open)
  TextRobot     implements Robot via TryFrom<&str/String>

dist_maze/      Distributed layer (WIP — stubs only)
  DistMazeServer<M: Maze>   wraps any Maze, exposes state over TCP
  DistMazeClient            implements Maze by proxying to DistMazeServer over TCP
  DistRobot                 implements Robot using DistMazeClient

lib.rs          Cell enum (Open/Wall/Finish), Direction enum + reverse(), new_bot() factory
```

**Key invariant:** `Robot::go` takes `&self` (not `&mut self`). Interior mutability is achieved by `RobotInternal` wrapping the `Maze` in `RefCell<Box<dyn Maze>>`. New `Robot` implementors must follow this pattern and delegate to `RobotInternal` through `get_internal()`.

**Factory pattern:** `new_bot(maze_source)` is a convenience wrapper for `maze_source.try_into()`. Use it when the concrete robot type can be inferred.

## Distributed system design

Three node roles:

- **Maze node** — single node holding maze state; bots query it via TCP to ensure consistency
- **Bot nodes** — N nodes each running a robot binary; coordinate via UDP broadcast on a shared port
- **Monitor node** (optional) — listens on a second UDP port for log messages from all nodes

`DistMazeServer` + `DistMazeClient` implement the bot↔maze TCP channel. The bot↔bot swarm UDP layer (`Swarm` struct) is not yet implemented — see TODOs in `index.md`.

## Examples

- `examples/text_dfs.rs` — working DFS solver using `TextRobot`; has `#[cfg(test)]` cases runnable with `cargo test --example text-dfs`
- `examples/dist_maze.rs` — maze node entry point (stub, `todo!()`)
- `examples/dist_bot.rs` — bot node entry point (stub, `todo!()`)
- `examples/test-maze.txt` — sample maze file for manual runs
