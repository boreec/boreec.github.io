---
title: "devlog #3 - "
categories: [Pet Project]
tags: [bevy, devlog, ecs, roguelike, rust]
draft: false
---

1. [Migration: Bevy v0.12.1 to v0.13.1](#migration-bevy-v0121-to-v0131)
2. [Mechanics/System: Multiple maps](#mechanicssystem-multiple-maps)
    1. [Adding an exit tile](#adding-an-exit-tile)
    2. [Generating the exit position](#generating-the-exit-position)
    3. [Despawning entities on map exit](#despawning-entities-on-map-exit)
3. [Miscellaneous](#miscellaneous)
4. [Final result](#final-result)
5. [Closing thoughts](#closing-thoughts)

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

## Miscellaneous

Besides the sections above, here are some small changes that also occurred:

- `setup_game` was renamed `setup_main_camera` and moved to `src/camera/mod.rs`
- `src/map.rs` was moved to its own plugin in `src/map/mod.rs`
- although bevy v0.13.1 was released, there was another fix with v0.13.2 that
  I migrated to, but there was no breaking changes

## Final result

Here's a quick video showcasing all new features/mechanics/content/etc.

## Closing thoughts

As always, if you have any suggestions/remarks regarding the devlog content, the
code or anything else, please reach out! Don't hesitate to drop a &#11088; on the [project's page](https://github.com/boreec/roguelike).
