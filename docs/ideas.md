- probe "swarm" expansion pack for [andrew-chang-dewitt/maze-robot][1]

  can allow for the following possibilities:
  - multiplayer modes (two different programs solve the maze
    collaboratively, or adversarially)
  - scaling across massive mazes (e.g. loading zones in mmo games)
  - graduated introduction to parallelism & distributed programming
    (by providing very high-level abstractions, plus exposing more
    limited versions as mid- & low-level abstractions & asking user
    to implement their own abstractions as needed)

- distributed, offline-first data store
  - using crdts, like [vlcn-io/cr-sqlite][2]
  - or perhaps other ways, like [superfly/litefs][3] (read more about [how it works][4])

[1]: https://github.com/andrew-chang-dewitt/maze-robot
[2]: https://github.com/vlcn-io/cr-sqlite
[3]: https://github.com/superfly/litefs
[4]: https://fly.io/docs/litefs/how-it-works/

