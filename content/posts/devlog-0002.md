---
title: "devlog #2 - Map Switching System"
categories: [Pet Project]
tags: [bevy, devlog, ecs, roguelike, rust]
draft: true
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
written so far.

Anyway, enough smalltalk, let's dive into this blog entry. This month I've
especially been focusing on implementing a way for the character to leave and
enter maps with a few extra.

1. [Content: Settling on a color palette](#content-settling-on-a-color-palette)
1. [Content: A horse sprite](#content-a-horse-sprite)
2. [Final result](#final-result)
3. [Closing Thoughts](#closing-thoughts)

## Content: Settling on a color palette

So far, I haven't given much thought to the game's appearance. With the
increasing number of sprites, I decided it was time to establish a color
palette. The concept is to craft sprites using a limited set of colors that
harmonize well (similar to a painter restricting their palette).

To find a fitting color palette, I used [lospec.com]("https://lospec.com"),
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

## Content: A horse sprite

## Final result

## Closing thoughts
