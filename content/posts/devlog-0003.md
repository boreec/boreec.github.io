---
title: "devlog #3 - Switching between maps"
categories: [Pet Project]
tags: [bevy, devlog, ecs, roguelike, rust]
date: 2024-04-29
---

1. [Migration: Bevy v0.12.1 to v0.13.1](#migration-bevy-v0121-to-v0131)
2. [Mechanics/System: Multiple maps](#mechanicssystem-multiple-maps)
    1. [Adding an exit tile](#adding-an-exit-tile)
    2. [Generating the exit position](#generating-the-exit-position)
    3. [Despawning entities on map exit](#despawning-entities-on-map-exit)
    4. [Showing the map number](#showing-the-map-number)
3. [Miscellaneous](#miscellaneous)
4. [Final result](#final-result)
5. [Closing thoughts](#closing-thoughts)

You can still read the [previous devlog](/posts/devlog-0002) if you missed it.

## Migration: Bevy v0.12.1 to v0.13.1

As I mentioned in the previous post, Bevy upgraded to v0.13 with a lot of new
features (and breaking changes). Since then, the v0.13.1 was released with its
various fixes and several guides updated their documentation accordingly.

Hopefully, the code base did not need too much rework, it was mainly:

1. Renaming `Input` to `ButtonInput`
2. Renaming `add_state` to `init_state`
3. Using the winit 0.29 new key codes
4. `TextureAtlas` having separate layout and handle.

That last point was the trickiest, so let me share how the tileset resources
are now defined:

```rust
use bevy::asset::LoadedFolder;
use bevy::prelude::*;

use crate::prelude::*;

#[derive(Default, Resource)]
pub struct TilesetFolder(pub Handle<LoadedFolder>);

#[derive(Default, Resource)]
pub struct TilesetActor(pub Handle<TextureAtlasLayout>, pub Handle<Image>);

#[derive(Default, Resource)]
pub struct TilesetTerrain(pub Handle<TextureAtlasLayout>, pub Handle<Image>);

pub fn initialize_tileset_actor_resource(
    handle: &UntypedHandle,
    texture_atlases: &mut ResMut<Assets<TextureAtlasLayout>>,
    commands: &mut Commands,
) {
    let texture_atlas = TextureAtlasLayout::from_grid(
        Vec2::new(SPRITE_TILE_WIDTH, SPRITE_TILE_HEIGHT),
        TILESET_ACTOR_COLUMNS,
        TILESET_ACTOR_ROWS,
        None,
        None,
    );

    let atlas_handle = texture_atlases.add(texture_atlas);
    let img_handle: Handle<Image> = handle.clone().typed();
    commands.insert_resource(TilesetActor(atlas_handle, img_handle));
}

pub fn initialize_tileset_terrain_resource(
    handle: &UntypedHandle,
    texture_atlases: &mut ResMut<Assets<TextureAtlasLayout>>,
    commands: &mut Commands,
) {
    let texture_atlas = TextureAtlasLayout::from_grid(
        Vec2::new(SPRITE_TILE_WIDTH, SPRITE_TILE_HEIGHT),
        TILESET_TERRAIN_COLUMNS,
        TILESET_TERRAIN_ROWS,
        None,
        None,
    );
    let atlas_handle = texture_atlases.add(texture_atlas);
    let img_handle: Handle<Image> = handle.clone().typed();
    commands.insert_resource(TilesetTerrain(atlas_handle, img_handle));
}
```

The resources structs now contain the additional image handle as it has been
decoupled from its layout.

For using the tileset resources, the `SpriteSheetBundle` uses the `texture`
attribute for the image handle, the `sprite` attribute for the rendering
properties and the `atlas` attribute for the layout and the tile index:

```rust
pub fn initialize_rabbits(
    commands: &mut Commands,
    map: &Map,
    tileset: &TilesetActor,
) {
    for _ in 0..3 {
        let map_position = map.generate_random_spawning_position();
        let (sprite_x, sprite_y) = calculate_sprite_position(&map_position);
        commands.spawn(RabbitBundle {
            rabbit: Rabbit,
            position: map_position,
            sprite: SpriteSheetBundle {
                atlas: TextureAtlas {
                    layout: tileset.0.clone(),
                    index: TILESET_ACTOR_IDX_RABBIT,
                },
                transform: Transform::from_xyz(
                    sprite_x,
                    sprite_y,
                    Z_INDEX_ACTOR,
                ),
                texture: tileset.1.clone(),
                sprite: Sprite::default(),
                ..Default::default()
            },
        });
    }
```

In the case you are also in need to migrate, I recommend the following
resources:

- https://taintedcoders.com/bevy
- https://bevyengine.org/learn/migration-guides/0-12-to-0-13/

I compiled all the migration changes in a single
[pull request](https://github.com/boreec/roguelike/pull/1).

## Mechanics/System: Multiple maps

### Adding an exit tile

In a traditional roguelike, the character is not limited to one map, but moves
across many. The exit may look like a staircase, a door or simply an arrow on
the ground. So in order to implement this mechanic, the first step was to 
create a new tile representing the exit.

{{<
    figure 
    src="/img/blog/devlog/roguelike-0017.png"
    title="a sign post with an arrow indicates the map exit"
>}}

Code-wise, the tile is set as walkable so the player needs to be on the tile
to trigger the map change:

```rust
#[derive(Clone, Component)]
pub enum TileType {
    Grass,
    GrassWithFlower,
    GrassWithStone,
    LevelExit,
}

impl TileType {
    pub const fn to_sprite_idx(tile_type: &Self) -> usize {
        match tile_type {
            Self::Grass => TILESET_TERRAIN_IDX_GRASS,
            Self::GrassWithFlower => TILESET_TERRAIN_IDX_GRASS_WITH_FLOWER,
            Self::GrassWithStone => TILESET_TERRAIN_IDX_GRASS_WITH_STONE,
            Self::LevelExit => TILESET_TERRAIN_IDX_SIGNPOST,
        }
    }

    pub const fn is_walkable(&self) -> bool {
        match self {
            Self::Grass | Self::GrassWithFlower => true,
            Self::GrassWithStone => false,
            Self::LevelExit => true,
        }
    }
}
```

### Generating the exit position

A `Map` may have more than one exit, so I added a vector of `MapPosition` to
the struct:

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
    pub tiles: Vec<TileType>,
    /// The exits positions for the map.
    pub exits: Vec<MapPosition>,
}
```

For now, the exit position is generated randomly on the right side of the map.
There's no implementation of path validation yet, so it could happen that no
path exist between the player and the exit point.

```rust
impl Map {
    /// Adds an exit tile on the right side of the map. The position is
    /// selected randomly.
    pub fn add_exit_tile(&mut self) {
        let spawnable_positions: Vec<_> = self
            .tiles
            .iter()
            .enumerate()
            .filter(|(index, tile)| {
                tile.is_walkable() && *index % self.width == self.width - 1
            })
            .map(|(index, _)| index)
            .collect();

        if spawnable_positions.is_empty() {
            panic!("There are no spawnable positions");
        }

        let mut rng = rand::thread_rng();
        let index = *spawnable_positions.choose(&mut rng).unwrap();

        self.tiles[index] = TileType::LevelExit;

        let exit_position = MapPosition {
            x: index % self.width,
            y: index / self.width,
        };

        self.exits.push(exit_position);
    }
}
```

The function `add_exit_tile` needs to be added explicitly to the `from` trait
implementations:

```rust
impl From<CellularAutomaton> for Map {
    /// Constructs a `Map` using a cellular automaton.
    ///
    /// # Arguments
    ///
    /// - `ca`: A `CellularAutomaton` initialized struct.
    ///
    /// # Returns
    ///
    /// A `Map` where the tiles are determined by the cellular automaton state.    
    fn from(ca: CellularAutomaton) -> Self {
        let mut map = Self {
            width: ca.width,
            height: ca.height,
            tiles: ca
                .cells
                .iter()
                .map(|cellular_state| match cellular_state {
                    CellularState::Alive => TileType::GrassWithStone,
                    CellularState::Dead => TileType::Grass,
                })
                .collect(),
            exits: vec![],
        };
        map.add_exit_tile();
        map
    }
}

impl From<(PerlinNoise, usize, usize)> for Map {
    /// Constructs a `Map` using Perlin noise.
    ///
    /// # Arguments
    ///
    /// - `tuple`: A tuple with three parameters representing a `PerlinNoise` struct,
    ///            the width, and the height of the map.
    ///
    /// # Returns
    ///
    /// A `Map` where the tiles are determined by Perlin noise.
    fn from(tuple: (PerlinNoise, usize, usize)) -> Self {
        let mut cells: Vec<TileType> = Vec::new();
        for i in 0..tuple.1 {
            for j in 0..tuple.2 {
                let x_scaled = i as f64 * PERLIN_NOISE_SCALE;
                let y_scaled = j as f64 * PERLIN_NOISE_SCALE;
                let noise_value = tuple.0.perlin_noise(x_scaled, y_scaled);
                if noise_value > 2.2 {
                    cells.push(TileType::GrassWithFlower);
                } else if noise_value > -0.25 {
                    cells.push(TileType::Grass);
                } else {
                    cells.push(TileType::GrassWithStone);
                }
            }
        }

        let mut map = Self {
            width: tuple.1,
            height: tuple.2,
            tiles: cells,
            exits: vec![],
        };

        map.add_exit_tile();
        map
    }
}
```

### Despawning entities on map exit

First, we need to detect when the player is on an exit tile:

```rust
/// Checks if a player is on an exit tile. In that case, the game state is
/// switched to `GameState::CleanupMap`.
pub fn check_if_player_exit_map(
    query_map: Query<&Map>,
    query_player: Query<&MapPosition, With<Player>>,
    mut next_game_state: ResMut<NextState<GameState>>,
) {
    let map = query_map.single();
    let player_position = query_player.single();
    for exit_position in &map.exits {
        if player_position == exit_position {
            next_game_state.set(GameState::CleanupMap);
        }
    }
}
```

Before creating and switching to a new map, we need to clean up resources of
the map the player is going to exit. It means despawning entities of remaining
map actors, map tiles and the map itself. I added `GameState::CleanupMap` and
`GameState::CleanupActors` to that purpose:

```rust
/// States used exclusively during the game. It involves not only the map and
/// actors creation, but also the main game turn between the player and the
/// enemies.
///
/// The lifecycle of the game is:
/// 1. `Uninitialized` -> `InitializingMap`
/// 2. `InitializingMap` -> `InitializingActors`
/// 3. `InitializingActors` -> `PlayerTurn`
/// 4.
///   1. `PlayerTurn` -> `EnemyTurn`
///   2. `PlayerTurn` -> `CleanupActors`
/// 5.
///   1. `EnemyTurn` -> `PlayerTurn` (back to step 4.1)
///   2. `CleanupActors` -> `CleanupMap`
/// 6. `CleanupMap` -> `InitializingMap` (back to step 2)
#[derive(Debug, Clone, Copy, Default, Eq, PartialEq, Hash, States)]
pub enum GameState {
    /// Corresponds to the default state, before the game is running.
    #[default]
    Uninitialized,
    /// Corresponds to the map creation.
    InitializingMap,
    /// Corresponds to the creation of the map's actors.
    InitializingActors,
    /// Corresponds to the turn when the player can do a move or an action.
    PlayerTurn,
    /// Corresponds to the turn when the enemies can do a move or an action.
    EnemyTurn,
    /// Corresponds to the map cleanup (spawned entities removal).
    CleanupMap,
    /// Corresponds to the map's actors cleanup (spawned entities removal).
    CleanupActors,
}
```

So far, the actors were never *associated* to a specific map. But since we want
to cleanup and initialize actors on a specific map we need to make that link.
I created a new component named `MapNumber`. This component is used in addition
to a new resource `CurrentMapNumber`:

```rust
#[derive(Bundle)]
pub struct MapBundle {
    map: Map,
    map_number: MapNumber,
}

/// Represents a number to identity a map.
#[derive(Component)]
pub struct MapNumber(pub usize);
```

```rust
/// Represents the current map number. The map number is increased every time
/// the player exits to another map.
#[derive(Default, Resource)]
pub struct CurrentMapNumber(pub usize);
```

This number for cleaning up and initializing our entities:

```rust
/// Removes all entities (`Map`, `Tile`, etc) related to the current map.
pub fn cleanup_map(
    mut commands: Commands,
    query: Query<(Entity, &MapNumber), Or<(With<Map>, With<Tile>)>>,
    mut next_game_state: ResMut<NextState<GameState>>,
    mut current_map_number: ResMut<CurrentMapNumber>,
) {
    for (entity, map_number) in &query {
        if map_number.0 == current_map_number.0 {
            commands.entity(entity).despawn();
        }
    }
    next_game_state.set(GameState::InitializingMap);
    current_map_number.0 += 1;
}
```

```rust
/// Removes actors for the current map.
pub fn cleanup_actors(
    mut commands: Commands,
    query: Query<(Entity, &MapNumber)>,
    mut next_game_state: ResMut<NextState<GameState>>,
    current_map_number: Res<CurrentMapNumber>,
) {
    for (entity, map_number) in &query {
        if map_number.0 == current_map_number.0 {
            commands.entity(entity).despawn();
        }
    }
    next_game_state.set(GameState::CleanupMap);
}
```

For the initialization, we bundle the `MapNumber` component to the entities and
the value corresponds to the resource `CurrentMapNumber`'s value:

```rust
/// Initializes all actors for the current map.
pub fn initialize_actors(
    mut commands: Commands,
    query_map: Query<(&Map, &MapNumber)>,
    mut query_player_map_position: Query<&mut MapPosition, With<Player>>,
    tileset: Res<TilesetActor>,
    current_map_number: Res<CurrentMapNumber>,
    mut next_game_state: ResMut<NextState<GameState>>,
) {
    let mut current_map = None;
    for (map, map_number) in &query_map {
        if map_number.0 == current_map_number.0 {
            current_map = Some(map);
        }
    }

    let current_map = current_map.unwrap();
    initialize_rabbits(
        &mut commands,
        current_map,
        &tileset,
        current_map_number.0,
    );

    // initialize the player only if there's no player created
    let player_map_position = query_player_map_position.get_single_mut();
    if player_map_position.is_err() {
        initialize_player(
            &mut commands,
            current_map,
            &tileset,
            current_map_number.0,
        );
    } else {
        // if the player already exists, set a new spawn on the map
        let new_spawn = current_map.generate_random_spawning_position();
        *player_map_position.unwrap() = new_spawn;
    }
    next_game_state.set(GameState::PlayerTurn);
}
```

```rust
/// Initialize a map by spawning tile entities depending on the map dimensions,
/// the tile placement algorithm, etc.
/// Lastly, the map entity is spawned.
fn initialize_map(
    mut commands: Commands,
    mut game_next_state: ResMut<NextState<GameState>>,
    tileset: Res<TilesetTerrain>,
    current_map_number: Res<CurrentMapNumber>,
) {
    let m = if rand::thread_rng().gen_bool(0.5) {
        Map::from((PerlinNoise::new(), MAP_WIDTH, MAP_HEIGHT))
    } else {
        let mut ca = CellularAutomaton::new(MAP_WIDTH, MAP_HEIGHT, 0.5);
        for _ in 0..50 {
            ca.transition();
        }
        ca.smooth();
        Map::from(ca)
    };

    for (i, tile) in m.tiles.iter().enumerate() {
        let tile_position = MapPosition {
            x: i % m.width,
            y: i / m.width,
        };
        let (sprite_x, sprite_y) = calculate_sprite_position(&tile_position);
        commands.spawn(TileBundle {
            tile: Tile,
            r#type: tile.clone(),
            sprite: SpriteSheetBundle {
                transform: Transform::from_xyz(
                    sprite_x,
                    sprite_y,
                    Z_INDEX_TILE,
                ),
                sprite: Sprite::default(),
                texture: tileset.1.clone(),
                atlas: TextureAtlas {
                    layout: tileset.0.clone(),
                    index: TileType::to_sprite_idx(tile),
                },
                ..Default::default()
            },
            map_number: MapNumber(current_map_number.0),
            map_position: tile_position,
        });
    }

    commands.spawn(MapBundle {
        map: m,
        map_number: MapNumber(current_map_number.0),
    });

    game_next_state.set(GameState::InitializingActors);
}
```

### Showing the map number

The same logic used for displaying the turn number is used. A marker struct is
component is created and initialized with the resource value:

```rust
/// Marker component to represent the ui element to display the current turn
/// number.
#[derive(Component)]
pub struct UiCurrentTurnText;

/// Marker component to represent the ui element to display the current map
/// number.
#[derive(Component)]
pub struct UiCurrentMapText;

/// Creates components for the ui elements.
pub fn setup_ui(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    current_turn_number: Res<CurrentTurnNumber>,
    current_map_number: Res<CurrentMapNumber>,
) {
    commands.spawn((
        UiCurrentTurnText,
        TextBundle::from_section(
            format!("Turn {}", current_turn_number.0),
            TextStyle {
                font: asset_server.load("fonts/GABOED.ttf"),
                font_size: UI_TEXT_TURN_SIZE,
                color: UI_TEXT_TURN_COLOR,
            },
        ),
    ));

    commands.spawn((
        UiCurrentMapText,
        TextBundle::from_section(
            format!("Map {}", current_map_number.0),
            TextStyle {
                font: asset_server.load("fonts/GABOED.ttf"),
                font_size: UI_TEXT_TURN_SIZE,
                color: UI_TEXT_TURN_COLOR,
            },
        )
        .with_style(Style {
            position_type: PositionType::Absolute,
            top: Val::Px(0.0),
            right: Val::Px(0.0),
            ..default()
        }),
    ));
}
```

The text content is updated when a new map is initialized:

```rust
/// Updates the ui element which represents the current map.
pub fn update_ui_current_map_text(
    mut query: Query<&mut Text, With<UiCurrentMapText>>,
    current_map_number: Res<CurrentMapNumber>,
) {
    let mut text = query.single_mut();
    text.sections[0].value = format!("Map {}", current_map_number.0);
}
```

{{<
    figure 
    src="/img/blog/devlog/roguelike-0018.png"
    title="the first map is #0"
>}}

{{<
    figure 
    src="/img/blog/devlog/roguelike-0019.png"
    title="just before exiting map #0"
>}}

{{<
    figure 
    src="/img/blog/devlog/roguelike-0020.png"
    title="spawning on map #1"
>}}


## Miscellaneous

Besides the sections above, here are some small changes that also occurred:

- `setup_game` was renamed `setup_main_camera` and moved to `src/camera/mod.rs`
- `src/map.rs` was moved to its own plugin in `src/map/mod.rs`
- although bevy v0.13.1 was released, there was another fix with v0.13.2 that
  I migrated to, but luckily there was no breaking changes
- added [typos](https://github.com/crate-ci/typos) hook to the pre-commit

## Final result

Here's a quick video showcasing all new features/mechanics/content/etc.

{{< youtube iOdo2TWWXOk >}}

As you may have noticed, there are a few bugs occurring such as respawning on
a rabbit or a rabbit spawning on a rock.

## Closing thoughts

Again, the progress may not look like much, but I'm happy to have the map
switching mechanic done. It made me reconsider how the actors are associated to
their respective maps. We could use that for generating several maps at once
instead of doing it when the player is on an exit tile.

As always, if you have any suggestions/remarks regarding the devlog content, the
code or anything else, please reach out! Don't hesitate to drop a &#11088; on
the [project's page](https://github.com/boreec/roguelike).

The project is growing and it's really exciting, thank you everyone.

