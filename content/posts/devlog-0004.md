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
important as it resulted in a bug where actors are left over the new map and
possibly overlap with other actors or non-walkable tiles.

{{<
    figure 
    src="/img/blog/devlog/roguelike-0022.png"
    title="the player is overlapping with the rabbit"
>}}

{{<
    figure 
    src="/img/blog/devlog/roguelike-0021.png"
    title="the rabbits spawning on non-walkable tiles"
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

I won't go into code details because I've went through big refactors on how the
actors are associated with map tiles (see following section)

## Refactor: Actors' positions owned by Map

Previously, I created a component `MapNumber` for all actors and map components
in order to link how they are connected. The problem with this approach is that
it makes queries chunkier and overcomplex. For example:

```rust
/// Checks if the player receives a directional input (i.e. an arrow key or a
/// WSQD key pressed), and moves the `Player` position accordingly.
pub fn check_player_directional_input(
    mut next_state: ResMut<NextState<GameState>>,
    mut query_player: Query<&mut MapPosition, With<Player>>,
    query_actors: Query<
        (&MapPosition, &MapNumber),
        (With<Actor>, Without<Player>),
    >,
    query_map: Query<(&Map, &MapNumber)>,
    input: Res<ButtonInput<KeyCode>>,
    current_map_number: Res<CurrentMapNumber>,
) {
    let mut player_pos = query_player.single_mut();

    let map = {
        let mut map_found = None;
        for (m, m_number) in &query_map {
            if m_number.0 == current_map_number.0 {
                map_found = Some(m);
            }
        }
        match map_found {
            Some(m) => m,
            None => {
                panic!(
                    "no map found to check for the directional player input"
                );
            }
        }
    };

    let occupied_pos: Vec<MapPosition> = query_actors
        .iter()
        .filter(|(_, m_n)| m_n.0 == current_map_number.0)
        .map(|(p, _)| p)
        .cloned()
        .collect();

    if input.any_just_pressed([KeyCode::ArrowRight, KeyCode::KeyD])
        && can_move_right(&player_pos, map, &occupied_pos)
    {
        move_right(&mut player_pos);
        next_state.set(GameState::EnemyTurn);
    }

    if input.any_just_pressed([KeyCode::ArrowLeft, KeyCode::KeyA])
        && can_move_left(&player_pos, map, &occupied_pos)
    {
        move_left(&mut player_pos);
        next_state.set(GameState::EnemyTurn);
    }

    if input.any_just_pressed([KeyCode::ArrowUp, KeyCode::KeyW])
        && can_move_up(&player_pos, map, &occupied_pos)
    {
        move_up(&mut player_pos);
        next_state.set(GameState::EnemyTurn);
    }

    if input.any_just_pressed([KeyCode::ArrowDown, KeyCode::KeyS])
        && can_move_down(&player_pos, map, &occupied_pos)
    {
        move_down(&mut player_pos);
        next_state.set(GameState::EnemyTurn);
    }
}
```


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

