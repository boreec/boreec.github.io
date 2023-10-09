---
icon: fa-solid fa-folder-tree
order: 1
---

<p align="justify">This page is a collection of several projects I made either
at the university or by myself. Most of them do not have other purpose than my
own enjoyment of perfecting my computer skills, learning new technologies and
concepts. Code and software quality vary from project to project, as some of
them were created a long time ago &#128116;.</p>

<p align="justify">All projects here are 100% mine, but as long you credit me
you can reuse the source code for your own purposes.</p>

1. [Cellular Automata](#cellular-automata)
    1. [Brian's Brain](#brians-brain)
    2. [Game of Life](#game-of-life)
2. [Games](#games)
    1. [Pong](#pong)
    2. [Snake](#snake)
    3. [Space Race](#space-race)
    4. [Tetris](#tetris)
3. [Web Services](#web-services)
    1. [Rest API](#rest-api)
    2. [URL Aliaser](#url-aliaser)

## Cellular Automata

### Brian's Brain

According to [Wikipedia](https://en.wikipedia.org/wiki/Brian's_Brain):
> Brian's Brain is a cellular automaton devised by Brian Silverman, which is 
very similar to his Seeds rule. 

<p align="justify">
I originally made the project because I was interested in learning Vulkan while
improving my skills in Rust. The hardest part was to set up the graphics
pipeline and decomposing the shapes in vertices that can be processed by the
GPU.</p>

- **Tech Stack:** Rust, Vulkan
- **Source Code:** [GitHub](https://gitlab.com/boreec/brian-s-brain), [GitLab](https://github.com/boreec/Brian-s-Brain)
- **Demo:** [Video](https://www.youtube.com/watch?v=r0fTs15-Qg0)

### Game of Life

According to [Wikipedia](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life):
> The Game of Life, also known simply as Life, is a cellular automaton devised
by the British mathematician John Horton Conway in 1970. It is a zero-player
game,meaning that its evolution is determined by its initial state, requiring
no further input. One interacts with the Game of Life by creating an initial
configuration and observing how it evolves. It is Turing complete and can 
simulate a universal constructor or any other Turing machine. 

<p align="justify">This one is probably a classic. I wrote many versions in
several languages, but for some reason I only managed to find the one I made
during my multi-agent systems class at university while we were learning
NetLogo.</p>

- **Tech Stack:** NetLogo
- **Source Code:** [GitLab](https://gitlab.com/boreec/gol_1)

## Games

### Pong

![Game view](https://raw.githubusercontent.com/boreec/Pong/master/img/game.png)

According to [Wikipedia](https://en.wikipedia.org/wiki/Pong):
> Pong is a table tennis–themed twitch arcade sports video game, featuring
simple two-dimensional graphics, manufactured by Atari and originally released
in 1972.

<p align="justify">I made a reproduction of the popular Atari's arcade game.
This version is simplified as the ball's bounce goes only in 8 possible
directions (horizontal, vertical and diagonal).</p>

- **Tech Stack:** Rust, SDL2
- **Source Code:** [GitHub](https://github.com/boreec/Pong), [GitLab](https://gitlab.com/boreec/pong)
- **Demo:** [Video](https://www.youtube.com/watch?v=FyqXscHFBu0)

### Snake

![Game View](https://gitlab.com/boreec/snake/-/raw/master/img/snake.png)

According to [Wikipedia](https://en.wikipedia.org/wiki/Snake_(video_game_genre)):
> Snake is a sub-genre of action video games where the player maneuvers the end
of a growing line, often themed as a snake. The player must keep the snake from
colliding with both other obstacles and itself, which gets harder as the snake
lengthens.

<p align="justify">Probably with Pong, one of the most reproduced games ever.
This one is probably my very first game written in Rust. The interface is
really basic, not to say crude. I'm still quite proud of this milestone as a
software developer.</p>

- **Tech Stack:** Rust, SDL2
- **Source Code:** [GitHub](https://github.com/boreec/Snake), [GitLab](https://gitlab.com/boreec/snake)
- **Demo:** [Video](https://www.youtube.com/watch?v=Heaoez-ZWxA)

### Space Race

![Game View](https://gitlab.com/boreec/space-race/-/raw/master/asset/img/game.png)

According to [Wikipedia](https://en.wikipedia.org/wiki/Space_Race_(video_game))
> Space Race is an arcade game developed by Atari, Inc. and released on July 
16, 1973. It was the second game by the company, after Pong (1972), which
marked the beginning of the commercial video game industry. In the game, two
players each control a rocket ship, with the goal of being the first to move
their ship from the bottom of the screen to the top. Along the way are
asteroids, which the players must avoid. Space Race was the first racing arcade
video game and the first game with a goal of crossing the screen while avoiding
obstacles. 

<p align="justify">Probably the most advanced game I`ve reproduced with Rust
and SDL2. I enjoyed doing it as it really differs from the game projects people
usually undertake to improve their programming skills.</p>

- **Tech Stack:** Rust, SDL2
- **Source Code:** [GitHub](https://github.com/boreec/Space-Race), [GitLab](https://gitlab.com/boreec/space-race/)
- **Demo:** [Video](https://www.youtube.com/watch?v=bzm3udWB7Kc)

### Tetris

According to Wikipedia:
> In Tetris, players complete lines by moving differently shaped pieces
(tetrominoes), which descend onto the playing field. The completed lines
disappear and grant the player points, and the player can proceed to fill the
vacated spaces. The game ends when the uncleared lines reach the top of the
playing field. The longer the player can delay this outcome, the higher their
score will be. In multiplayer games, players must last longer than their
opponents

<p align="justify">I had some difficulties at first using Qt logic for
rendering the game in an OpenGL component within the game loop. This version
is very basic, but it's working &#128513;.</p>

- **Tech Stack:** C++14, CMake, Qt
- **Source Code:** [GitHub](https://github.com/boreec/Tetris), [GitLab](https://gitlab.com/boreec/tetris) 
- **Demo:** [Video](https://www.youtube.com/watch?v=kj1cXrnWwcY)

## Web services

### Rest API

<p align="justify">I wrote a REST API as an assesmsent for a job interview.
Since I spent quite some time working on it, I thought about including it here.
This API provides a few endpoints for dealing with a database of people.</p>

- **Tech Stack:** Flask, Python, SQLite
- **Source Code:** [GitHub](https://github.com/boreec/REST-API)

### URL Aliaser

<p align="justify">This application serves as a user-friendly URL aliasing
server. Users can provide long URLs, and the server generates shortened aliases
that redirect to the original URLs. The aliases act as convenient shortcuts for
accessing the original links.</p>

- **Tech Stack:** Go
- **Source Code:** [GitHub](https://github.com/boreec/url-aliaser)