---
title: "devlog #0 - A Roguelike for 2024 ?"
date: 2023-12-29
categories: [Pet Project]
tags: [bevy, devlog, ecs, roguelike, rust]
---

## Introduction

Over the last two months, I've been hooked on
[Tales of Maj'Eyal](https://en.wikipedia.org/wiki/Tales_of_Maj%27Eyal), playing
it almost on a daily basis. Developed in 2012, its gameplay and engine derive 
directly from classical roguelikes, particularly
[Angband](https://en.wikipedia.org/wiki/Angband_(video_game)) which is itself
based on [Moria](https://en.wikipedia.org/wiki/Moria_(1983_video_game)).

The game is quite straightforward – you guide a top-down character through a 2D
world. Navigating randomly generated levels, your goal is to defeat the final
boss. Despite its apparent simplicity, its richness lies in the sheer number of 
playable classes and races, its difficulty modes and its various campaigns 
(some DLCs, arena mode, infinite dungeon mode, etc).

{{< figure src="/img/blog/devlog/tome4-character-creation.png" 
    title="You can play up to 37 classes and 16 races"
>}}

{{< figure src="https://i.redd.it/5aftw4rj48n11.png" 
    title="The game consists of inanimated sprites and an UI similar to many RPGS"
>}}

I won't delve more into that game, but I highly suggest checking it out. While
playing, I pondered the idea of creating my own roguelike game, drawing 
inspiration primarily from it, but not only as I'm an avid player of related 
games. Here's a list I recommend:

- [Dofus](https://en.wikipedia.org/wiki/Dofus)
- [Dwarf Fortress](https://en.wikipedia.org/wiki/Dwarf_Fortress)
- [Final Fantasy Tactics Advance](https://en.wikipedia.org/wiki/Final_Fantasy_Tactics_Advance)
- [Into the Breach](https://en.wikipedia.org/wiki/Into_the_Breach)
- [NetHack](https://en.wikipedia.org/wiki/NetHack)
- [Triangle Strategy](https://en.wikipedia.org/wiki/Triangle_Strategy)

## Project Overview

The project's scope isn't entirely clear at the moment, but my primary aim is to
explore the extent to which I can develop my own roguelike. As it's currently in
the prototype phase, there's no set plan for a release.

In terms of the tech stack, I went for [Rust](https://www.rust-lang.org/) and
[Bevy](https://bevyengine.org/) for a few reasons:
1. I've been eager to learn more about [Bevy](https://bevyengine.org/) and the 
[ECS paradigm](https://en.wikipedia.org/wiki/Entity_component_system) since my
[last silly pet project](https://boreec.github.io/posts/a-silly-project-for-halloween/).
2. Many popular game engines come with intricate UIs which require time to
get used to.
3. Distributing the game could become an issue, as seen in the recent
[Unity controversy](https://en.wikipedia.org/wiki/Unity_(game_engine)#Licensing).


## About the devlog

The idea with this devlog is to showcase recent changes made to the game. I
think it can be interesting for others to see the process I am going through for
creating a game and how new content/features are added. That being said, I won't
go too deep in the code itself, but I'll try to provide links to the GitHub
repository for the interesting parts.

This first entry is a little bit special. I didn't want to start the devlog
completely from scratch since I felt it would lack actual content to talk
about. Therefore, I dedicated the month of December to building a first version
that can be expanded upon (Hence the #0 for the entry). This way, I can tackle
2024 with a very fresh perspective! Regarding the posting frequency, I'll try
to post once a month, but I can't promise anything.

## First version

In every project, the most difficult part is the first step. Aiming too big from
the start is the best way to give up, so I aimed very small.

My very first objective was to display a tileset image in a window. I spent
about 5-10 minutes crafting 3 rather rudimentary 64x64px tiles in Gimp,
arranging them into a single image (a [tileset](https://en.wikipedia.org/wiki/Tile-based_video_game)).
Hopefully, Bevy provides its own
[TextureAtlas](https://docs.rs/bevy/latest/bevy/sprite/struct.TextureAtlas.html)
for working with this kind of resources
(see [related commit](https://github.com/boreec/roguelike/tree/5ca14e73de063356d455661970db60c8b8f9ff9b)).

{{< figure src="/img/blog/devlog/roguelike-0001.png" 
    title="Step 1: Display a tileset image"
>}}

For the next step, I created a first 10x10 map by spawning tile entities (with
Bevy's [SpriteSheetBundle](https://docs.rs/bevy/latest/bevy/prelude/struct.SpriteSheetBundle.html)).
The map was not displayed properly as Bevy's coordinates origin is the middle of
the screen (see [related commit](https://github.com/boreec/roguelike/tree/603d43d8f4a5a91152a0b1a8c32b298758562867)).

{{< figure src="/img/blog/devlog/roguelike-0002.png" 
    title="Step 2: Display individual tiles, forming a 10x10 map"
>}}

I fixed the rendering by shifting all tiles in relation with the origin. This
[blog post](https://www.mikechambers.com/blog/2022/10/29/understanding-the-2d-coordinate-system-in-bevy/)
written by Mike Chambers details how Bevy's coordinates system work. Besides the
map, there's also the character's tile showing up (see [related commit](https://github.com/boreec/roguelike/tree/adbc39c47a4fab1f6fafcbbbfe066a93065355c2)).

{{< figure src="/img/blog/devlog/roguelike-0003.png" 
    title="Step 3: Fix the map rendering and display character sprite"
>}}

The next logical step is to make the character evolves in its environment. I
created a new tile to depict a stone, which functions as an impassable obstacle.
Moreover, I implemented keyboard input capture for movement control, allowing
the player to move in the specified direction. It includes obstacle detection,
considering elements like stones or map borders (see [related commit](https://github.com/boreec/roguelike/tree/54658bfeecb5f5a1d8b81280d6bb516f51b1cf91)).

{{< figure src="/img/blog/devlog/roguelike-0004.png" 
    title="Step 4: The character moving around in the environment"
>}}

I also took advantagee of Bevy's [states](https://docs.rs/bevy/latest/bevy/prelude/struct.State.html),
which allows to execute some systems only when a state is active. I defined two
type of states:

- **Application States**: They differentiate between the loading assets state
and the in-game state. This separation is crucial as there's no need to run the
game until all assets are fully loaded.
- **Game States**: They are used for differentiating between various game
phases, including level initialization, player turns, enemmy turns, etc.

```rust
use bevy::prelude::*;

#[derive(Debug, Clone, Copy, Default, Eq, PartialEq, Hash, States)]
pub enum AppState {
    #[default]
    LoadingAssets,
    InGame,
    Finished,
}

#[derive(Debug, Clone, Copy, Default, Eq, PartialEq, Hash, States)]
pub enum GameState {
    #[default]
    Uninitialized,
    Initializing,
    PlayerTurn,
    EnemyTurn,
}
```

Moving from one state to another is also trivial using the `set` method, for
example once all assets are loaded:
```rust
fn check_assets(
    mut app_next_state: ResMut<NextState<AppState>>,
    mut game_next_state: ResMut<NextState<GameState>>,
    mut events: EventReader<AssetEvent<LoadedFolder>>,
) {
    for event in events.read() {
        match event {
            AssetEvent::LoadedWithDependencies { id: _ } => {
                println!("asset loaded!");
                app_next_state.set(AppState::InGame);
                game_next_state.set(GameState::Initializing);
            }
            _ => {}
        }
    }
}
```

Finally, the last worth mentioning things I added to this version are:

- zoom-in/zoom-out using the mouse wheel
- map's grid displaying when pressing G
- current's turn displaying

{{< figure src="/img/blog/devlog/roguelike-0005.png" 
    title="Step 5: Display the turn number and the grid"
>}}

## Final result

Here's a quick video showcasing all mechanics and features.

{{< youtube A49WU099Igs >}}

The source code is available on [GitHub](https://github.com/boreec/roguelike).

## Closing Thoughts

If you want to support the project, drop a ⭐ on [GitHub](https://github.com/boreec/roguelike),
which can directly make me stop procrastinating. Also, writing about my
projects is still a new thing for me, so if you think I can improve in some way,
please reach out!

Happy new year 2024!
