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

## contents

- (a) what it is
- (b) language(s) & dependencies
- (c) system design
- (d) example inputs & expected outputs
- stretch goals

## (a) what it is

- probe "swarm" expansion pack for [andrew-chang-dewitt/maze-robot][1]

## (b) language(s) & dependencies

## (c) system design

## (d) example inputs & expected outputs

- basic dfs of `N` bots solving the same test maze as the existing dfs
  test solution; must use high-level abstractions the project will
  provide
- multi-pronged approach, where the swarm divides & has some bots try
  one approach, while others try another & all share what they've
  discovered so far
- multiplayer modes (N different programs solve the maze
  collaboratively, or adversarially, (or both?))

## stretch goals

- scaling across massive mazes (e.g. loading zones in mmo games)
- graduated introduction to parallelism & distributed programming
  (by providing very high-level abstractions, plus exposing more
  limited versions as mid- & low-level abstractions & asking user
  to implement their own abstractions as needed)

[1]: https://github.com/andrew-chang-dewitt/maze-robot
