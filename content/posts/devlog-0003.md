---
title: "devlog #3 - "
categories: [Pet Project]
tags: [bevy, devlog, ecs, roguelike, rust]
draft: false
---

## Migration: Bevy v0.12.1 to v0.13.1

As I mentioned in the previous post, Bevy upgraded to v0.13 with a lot of new
features (and breaking changes). Since then, the v0.13.1 was released with its
various fixes and several guides updated their documentation to cover the
changes.

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

Now for using the tileset resources, the `SpriteSheetBundle` uses the `texture`
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

Here are the resources that I found helpful for the migration:

- https://taintedcoders.com/bevy
- https://bevyengine.org/learn/migration-guides/0-12-to-0-13/

## Final result

Here's a quick video showcasing all new features/mechanics/content/etc.

## Closing thoughts

As always, if you have any suggestions/remarks regarding the devlog content, the
code or anything else, please reach out! Don't hesitate to drop a &#11088; on the [project's page](https://github.com/boreec/roguelike).
