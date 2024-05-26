---
title: "devlog #4 - NPC random movements"
categories: [Pet Project]
tags: [bevy, devlog, ecs, roguelike, rust]
draft: false
---

This is already the fifth blog post about this project! It's probably the
longest I've ever worked on a single one. For this entry, I've dealt with a
lot of refactoring, addressed code smells, and set up the fundamentals of
what will become pathfinding.

1. [Bug Fixing: Actors overlapping](#bug-fixing-actors-overlapping-and-on-non-walkable-tiles)
2. [Refactor: Actors positions owned by Map](#refactor-actors-positions-owned-by-map)
3. [Mechanics/System: Actors movement](#mechanicssystem-actors-movement)
    1. [Random movements](#random-movements)
    2. [Straight forward movements](#straight-forward-movements)
4. [Miscellaneous](#miscellaneous)
5. [Final result](#final-result)
6. [Closing thoughts](#closing-thoughts)

You can still read the [previous devlog](/posts/devlog-0003) if you missed it.

## Bug Fixing: Actors overlapping and on non-walkable tiles

Last month, the map and actors cleanup were introduced when the player leaves
to another map. I didn't notice at the time, but the order of operations is
important as it resulted in a bug where the actors can overlap.

{{<
    figure 
    src="/img/blog/devlog/roguelike-0022.png"
    title="the player is overlapping with the rabbit"
>}}

The solving involved doing this order of operations:

1. Initialize Map
2. Initialize Actors
3. Cleanup Actors
4. Cleanup Map

Instead of:

1. Initialize Map
2. Initialize Actors
3. Cleanup Map
4. Cleanup Actors

## Bug Fixing: Actors spawning on non-walkable tiles

{{<
    figure 
    src="/img/blog/devlog/roguelike-0021.png"
    title="the player is overlapping with the rabbit"
>}}

## Refactor: Actors' positions owned by Map

## Mechanics/System: Actors movement

### Random movements

### Straight forward movements

## Miscellaneous

## Final result

## Closing thoughts

As always, if you have any suggestions/remarks regarding the devlog content, the
code or anything else, please reach out! Don't hesitate to drop a &#11088; on
the [project's page](https://github.com/boreec/roguelike).

The project is growing and it's really exciting, thank you everyone.

