---
title: "devlog #2 - Palette and Tileset update"
categories: [Pet Project]
tags: [bevy, devlog, ecs, roguelike, rust]
date: 2024-02-28
---


If you've read the [devlog entry](/posts/devlog-0001) that I released in
January, you might have observed a change in the website's appearance since
then. Indeed, earlier the site images had a persistent shimmer, which appeared
to be a recurring issue with Jekyll's
[Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy). While there might
be a simple fix that I might have missed, I opted to switch to a complete 
different theme in the end!

The current website is now generated using [hugo](https://gohugo.io/), a really
fast and lightweight framework for building static website. Migrating the blog 
posts and pages was a swift process, given the limited amount of content I've
written so far. If you still see the old theme on other pages, clear the cache
for this website in your web browser.

Anyway, let's dive into the devlog. This month was quite light in terms of
progress, but a small step is still a step!

1. [Content: Settling on a color palette](#content-settling-on-a-color-palette)
2. [Chore: Dividing the main tileset](#chore-dividing-the-main-tileset)
3. [News: Bevy 0.13](#news-bevy-013)
4. [Final result](#final-result)
5. [Closing Thoughts](#closing-thoughts)

## Content: Settling on a color palette

So far, I haven't given much thought to the game's appearance. With the
increasing number of sprites, I decided it was time to establish a color
palette. The concept is to craft sprites using a limited set of colors that
harmonize well (similar to a painter restricting their palette).

To find a fitting color palette, I used [lospec.com](https://lospec.com),
which is a website where people post their color palette. I found one named
[RESURRECT 64](https://lospec.com/palette-list/resurrect-64), by Kerrie Lake.
Applying this palette to the tileset has toned down the game's look for sure.

{{<
    figure 
    src="/img/blog/devlog/roguelike-0014.png"
    title="before using a color palette"
>}}


{{<
    figure 
    src="/img/blog/devlog/roguelike-0015.png"
    title="after using a color palette"
>}}

## Chore: Dividing the main tileset

So far, all tiles were put in the same tileset. It works great but it lacks
flexibility for future content. For example, it would make more sense to have
one tileset for the characters, one for monsters, one for terrain, etc.

So I split the main tileset in two, one for the terrain and one for the actors.

{{<
    figure 
    src="/img/blog/devlog/roguelike-0016.png"
    title="two tilesets"
>}}

Code-wise, each tileset has its own corresponding resource struct and
initializing method. Each tileset is loaded to its struct according to its
path:

```rust
use bevy::asset::LoadedFolder;
use bevy::prelude::*;

use crate::prelude::*;

#[derive(Default, Resource)]
pub struct TilesetFolder(pub Handle<LoadedFolder>);

#[derive(Default, Resource)]
pub struct TilesetActor(pub Handle<TextureAtlas>);

#[derive(Default, Resource)]
pub struct TilesetTerrain(pub Handle<TextureAtlas>);

pub fn initialize_tileset_actor_resource(
    handle: &UntypedHandle,
    texture_atlases: &mut ResMut<Assets<TextureAtlas>>,
    commands: &mut Commands,
) {
    let texture_atlas = TextureAtlas::from_grid(
        handle.clone().typed::<Image>(),
        Vec2::new(SPRITE_TILE_WIDTH, SPRITE_TILE_HEIGHT),
        TILESET_ACTOR_COLUMNS,
        TILESET_ACTOR_ROWS,
        None,
        None,
    );
    let atlas_handle = texture_atlases.add(texture_atlas);
    commands.insert_resource(TilesetActor(atlas_handle));
}

pub fn initialize_tileset_terrain_resource(
    handle: &UntypedHandle,
    texture_atlases: &mut ResMut<Assets<TextureAtlas>>,
    commands: &mut Commands,
) {
    let texture_atlas = TextureAtlas::from_grid(
        handle.clone().typed::<Image>(),
        Vec2::new(SPRITE_TILE_WIDTH, SPRITE_TILE_HEIGHT),
        TILESET_TERRAIN_COLUMNS,
        TILESET_TERRAIN_ROWS,
        None,
        None,
    );
    let atlas_handle = texture_atlases.add(texture_atlas);
    commands.insert_resource(TilesetTerrain(atlas_handle));
}
```



## News: Bevy 0.13

Bevy's new version 0.13 was released on [February 17th](https://bevyengine.org/news/bevy-0-13/).
It includes a ton of exciting features for this project, such as:
- Primitive Shapes
- Texture Slicing and Tiling
- Granular asset loading support
- New TextureAtlas object (simpler!)

Hopefully there's a [migration guide](https://bevyengine.org/learn/migration-guides/0-12-to-0-13/)
available to see how to update. I checked the impacted code and it seems that
only the inputs and the texture atlas would have to be rewritten.

I will delay the migration to later though as time is running out for this
month.

## Final result

Here's a quick video showcasing all new features/mechanics/content/etc.

{{< youtube 4ikroHbs82w >}}

## Closing thoughts

That's it for February! I hope you are not disappointed with the slow progress.
This month was quite busy for me and it was hard to find spare time to work on
this project, but I would be lying if I said that it's only a time issue. I have
less motivation to keep working on it recently.

Because of that, I decided to not publish a devlog issue for next month, but 
probably for April!

As always, if you have any suggestions/remarks regarding the devlog content, the
code or anything else, please reach out! Don't hesitate to drop a &#11088; on the [project's page](https://github.com/boreec/roguelike).
