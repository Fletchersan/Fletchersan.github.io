---
layout: post
title:  "Asteroids"
date:   2023-01-12 10:40:00 +0530
categories: CS Python
---

I created a simple game using pygame, it's asteroids. It's been done to death a 100 times I know, but I learnt some stuff!

## Game Loop

This is the first time I encountered a pattern such as this, as it's rare to see an
infinite loop in something apart from games (although a webserver listener might be
another case).

This was used for everything, and I mean everything, from setting the framerate to 
updating and rendering the objects in the game.

Essentially this represented the core logic of the code.

```
while (true)
{
  processInput();
  update();
  render();
}

```

## Sprites and vectors

I kind of expected drawing a sprite would involve actually drawing lol (Although that is
also done). Although in this case I used pygames inbuilt draw functions to draw the 
sprites.

The player sprite was a triangle, that was drawn using some basic vector calculation.
I calculated the three edges from the center position.