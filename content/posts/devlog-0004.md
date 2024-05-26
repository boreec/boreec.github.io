---
title: "devlog #4 - NPC random movements"
categories: [Pet Project]
tags: [bevy, devlog, ecs, roguelike, rust]
draft: true
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

The following code is really different from last month, but now, the tiles are
managed by the Map, and the actor are managed by the tiles:

```rust
/// Represents a tile.
#[derive(Clone, Component, Copy)]
pub struct Tile {
    pub kind: TileKind,
    pub actor: Option<Actor>,
}
```

```rust
/// Represents the environment where the actors interact together. A map is
/// made of tiles which has different properties for the actors.
#[derive(Component)]
pub struct Map {
    /// The map's width.
    pub width: usize,
    /// The map's height.
    pub height: usize,
    /// All tiles for the map, the vector index corresponds to the tile
    /// coordinates.
    pub tiles: Vec<Tile>,
    /// The exits positions for the map.
    pub exits: Vec<MapPosition>,
}
```

Additionally, a component `OnDisplay` is used to differentiate between the map
and actors that are on the screen with the ones that are kept in memory
(although there's no such thing for now):

```rust
/// Represents an entity that is on the screen and displayable by the camera.
#[derive(Component)]
pub struct OnDisplay;
```

It makes the queries simpler:

```rust
pub fn check_player_move_via_keys(
    mut next_state: ResMut<NextState<GameState>>,
    mut q_actors: Query<(&mut MapPosition, &Actor), With<OnDisplay>>,
    mut q_map: Query<&mut Map, With<OnDisplay>>,
    input: Res<ButtonInput<KeyCode>>,
) {
    let mut map = q_map.single_mut();

    let (mut pos_player, _) = q_actors
        .iter_mut()
        .filter(|(_, a)| a.is_player())
        .last()
        .expect("no player pos found");

    let pos_player_old = pos_player.clone();

    if input.any_just_pressed(KEYS_PLAYER_MOVE_RIGHT)
        && can_move_right(&pos_player.clone(), &map)
    {
        move_right(&mut map, &mut pos_player).unwrap();
    }

    if input.any_just_pressed(KEYS_PLAYER_MOVE_LEFT)
        && can_move_left(&pos_player.clone(), &map)
    {
        move_left(&mut map, &mut pos_player).unwrap();
    }

    if input.any_just_pressed(KEYS_PLAYER_MOVE_UP)
        && can_move_up(&pos_player.clone(), &map)
    {
        move_up(&mut map, &mut pos_player).unwrap();
    }

    if input.any_just_pressed(KEYS_PLAYER_MOVE_DOWN)
        && can_move_down(&pos_player.clone(), &map)
    {
        move_down(&mut map, &mut pos_player).unwrap();
    }

    if pos_player_old != pos_player.clone() {
        next_state.set(GameState::EnemyTurn);
    }
}
```

## Mechanics/System: Actors spawning

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

