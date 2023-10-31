---
title: A silly project for Halloween
date: 2023-10-31
categories: [Pet Project]
tags: [bevy, ecs, halloween, minimp3, mp3, rust]
---

{: style="text-align:center"}
![halloween-1](/assets/img/blog/halloween/2022_02.jpg)
*Halloween Night Sky over the French Countryside*

While Halloween never reached the same level of popularity in France as
Christmas, I've always had a preference for the former over the latter. In the
past, we would decorate our countryside home with glowing, carved pumpkins,
creating a truly unique nighttime landscape.

{: style="text-align:center"}
![halloween-2](/assets/img/blog/halloween/2022_01.jpg)
*A typical carved pumpkin we used to make*

Although my childhood and the house I grew up in are now distant memories, the
emotions and the ambiance of Halloween continue to resonate in me as the
holiday comes every year.

## The project idea

Recently, I had some interest learning [Bevy](https://bevyengine.org/), a
trendy game engine written in Rust. Although it is usable not only for games,
its [ECS (Entities-Components-Systems)](https://en.wikipedia.org/wiki/Entity_component_system)
makes it really easy to get started with 2D and 3D real-time projects. With Bevy
in my mind, and Halloween coming by, I wanted to create a small related project
to learn more about it.

At first, I didn't have a specific plan in mind, but I decided to draw a simple
ghost using [Bevy's built-in shapes](https://bevyengine.org/examples/2D%20Rendering/2d-shapes/).
As I worked on it, I noticed that I was making a similar mistake to what I had
done when I reproduced [Atari's Space Race](https://boreec.github.io/projects/#space-race)
game, where I initially drew everything with basic shapes and then had to
meticulously organize them in code.

{: style="text-align:center"}
![Game View](https://gitlab.com/boreec/space-race/-/raw/master/asset/img/game.png)
*Atari Space Race game reproduction*

Instead, I decided to save development time by using a sprite approach, so I
went with a Jack Skellington's head PNG. Scrolling through other Bevy examples,
I found one showcasing [how to scale a 3D object over time](https://bevyengine.org/examples/Transforms/scale/).
Seeing this, I thought, "What if the image scaled with a music's loudness?" The
idea is simple: as the music gets louder, the image grows larger and as the
music quiets down, the image shrinks.

## End result

<iframe width="560" height="315" src="https://www.youtube.com/embed/JpcI-rEiN2M?si=0gdT3bb4fo1EhZ5t&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

The source code is available [here](https://github.com/boreec/halloween_2023).

## What I learned

- ECS concept
- Bevy main framework
- minimp3 and the mp3 format

## What I struggled with

The following points are to be taken with a grain of salt as I am not a Bevy nor
a WASM expert. There are probably easy ways to come around these issues, but due
to time constraints I did not want to bother too much.

- Building the project in WASM and making it running in the current web page was
  part of my initial goal, but I could not compile to that target with minimp3.
- I did not understand how to easily specify dynamic path (for example through a
  CLI option) for resources as Bevy assumes that everything is in the assets
  folder.
- I am not sure how to make the image scale in a more esthetic way. I thought
  about using a low-pass filter on the music loudness level, but I did not have
  enough time to implement.
