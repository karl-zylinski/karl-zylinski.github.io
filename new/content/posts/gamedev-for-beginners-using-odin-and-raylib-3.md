---
title: "Make games using Odin and Raylib #3: An animated player"
date: 2024-02-20

draft: true

cover:
  image: "/odinraylib3/cover.png"
---

Today we will make our game look less dull by adding a run animation to the player character.

As usual, this post has a companion video, it says mostly the same stuff, but in video form. Beginners are encouraged to both watch and read, as it will make everything easier to understand.

<figure>
{{<youtube PUT HERE>}}
<figcaption>The companion video for this post. It contains mostly the same information. It can be helpful if you get confused by anything.</figcaption>
</figure>

## The code so far

This is how the game's code looked at the end of [part 2]({{< ref "/posts/gamedev-for-beginners-using-odin-and-raylib-2" >}}):

```C
package game

import rl "vendor:raylib"

main :: proc() {
    rl.InitWindow(1280, 720, "My First Game")
    player_pos := rl.Vector2 { 640, 320 }
    player_vel: rl.Vector2
    player_grounded: bool

    for !rl.WindowShouldClose() {
        rl.BeginDrawing()
        rl.ClearBackground(rl.BLUE)

        if rl.IsKeyDown(.LEFT) {
            player_vel.x = -400
        } else if rl.IsKeyDown(.RIGHT) {
            player_vel.x = 400
        } else {
            player_vel.x = 0
        }

        player_vel.y += 2000 * rl.GetFrameTime()

        if player_grounded && rl.IsKeyPressed(.SPACE) {
            player_vel.y = -600
            player_grounded = false
        }

        player_pos += player_vel * rl.GetFrameTime()

        if player_pos.y > f32(rl.GetScreenHeight()) - 64 {
            player_pos.y = f32(rl.GetScreenHeight()) - 64
            player_grounded = true
        }

        rl.DrawRectangleV(player_pos, {64, 64}, rl.GREEN)
        rl.EndDrawing()
    }

    rl.CloseWindow()
}
```

and this is how the game looks when run:

<figure>
<video autoplay loop muted width="100%"><source src="/odinraylib2/proper_jumping.mp4"></video>
<figcaption>How our game looked at the end of part 2. Today we will replace the green rectangle with a running cat animation</figcaption>
</figure>

## Replacing the green rectangle

Today we want to replace the green rectangle that represents the player character with some animated pixelart graphics.

First off, we need some picture to draw on the screen instead of the green rectangle. Let's use the run animation from my game [CAT &amp; ONION]():

<img src="/odinraylib3/cat_run.png" style="width: 100%; image-rendering: crisp-edges; image-rendering: pixelated; -ms-interpolation-mode: nearest-neighbor;" alt="a  four-frame sprite sheet of a cat running">

Right click the image it and save next to `my_first_game.odin` and make sure the name is `cat_run.png`.

As you see, this is a single image that contains four separate images of a cat running. Each one of those images is called an animation frame. We will make our game display one of them at a time in order to to make the player look like a running cat.

Just below the line `player_grounded: bool`, add this line:
```C
player_run_texture := rl.LoadTexture("cat_run.png")
```

This `LoadTexture` proc tells Raylib to load the image you just saved and downloaded. It will send the image to your graphics card (GPU), so that it is ready for being drawn. We can use the variable `player_run_texture` later to actually draw the texture.

> **NOTE**: What's the difference between an image and a texture? 