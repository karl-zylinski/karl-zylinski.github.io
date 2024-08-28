---
title: "Make games using Odin and Raylib #4: Switching animations"
date: 2024-03-19

draft: true

cover:
  image: "/odinraylib3/cover.png"
---

<figure>
{{<youtube qJA7laC3q18>}}
<figcaption>The companion video for this post. It contains mostly the same information. It can be helpful if you get confused by anything.</figcaption>
</figure>

## The code so far

<details>
<summary><strong>How the code looked at the end of part 3 (click to expand)</strong></summary>

```C
package game

import rl "vendor:raylib"

main :: proc() {
    rl.InitWindow(1280, 720, "My First Game")
    player_pos := rl.Vector2 { 640, 320 }
    player_vel: rl.Vector2
    player_grounded: bool
    player_flip: bool
    player_run_texture := rl.LoadTexture("cat_run.png")
    player_run_num_frames := 4
    player_run_frame_timer: f32
    player_run_current_frame: int
    player_run_frame_length := f32(0.1)
    
    for !rl.WindowShouldClose() {
        rl.BeginDrawing()
        rl.ClearBackground({110, 184, 168, 255})

        if rl.IsKeyDown(.LEFT) {
            player_vel.x = -400
            player_flip = true
        } else if rl.IsKeyDown(.RIGHT) {
            player_vel.x = 400
            player_flip = false
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

        player_run_width := f32(player_run_texture.width)
        player_run_height := f32(player_run_texture.height)

        player_run_frame_timer += rl.GetFrameTime()

        if player_run_frame_timer > player_run_frame_length {
            player_run_current_frame += 1
            player_run_frame_timer = 0

            if player_run_current_frame == player_run_num_frames {
                player_run_current_frame = 0
            }
        }

        draw_player_source := rl.Rectangle {
            x = f32(player_run_current_frame) * player_run_width / f32(player_run_num_frames),
            y = 0,
            width = player_run_width / f32(player_run_num_frames),
            height = player_run_height,
        }

        if player_flip {
            draw_player_source.width = -draw_player_source.width
        }

        draw_player_dest := rl.Rectangle {
            x = player_pos.x,
            y = player_pos.y,
            width = player_run_width * 4 / f32(player_run_num_frames),
            height = player_run_height * 4
        }

        rl.DrawTexturePro(player_run_texture, draw_player_source, draw_player_dest, 0, 0, rl.WHITE)
        rl.EndDrawing()
    }

    rl.CloseWindow()
}
```
</details>

and this is how the game looks when run:

<figure>
<video autoplay loop muted width="100%"><source src="/odinraylib3/animating_with_flip.mp4"></video>
<figcaption>How our game looked at the end of part 3. Today we will add another animation and switch between the two animations properly.</figcaption>
</figure>

## Creating a re-usable Animation struct

Currently the player's run animation is constantly playing, even when you stand still. This looks quite odd! Let's add a an animation for when the cat is standing still.

Adding another animation will be a bit messy unless we generalize our code a bit. We need to start off by taking all those variable named `player_run_...` and put them together into some reusable notion of an animation.

In order to do this we'll define a new `struct`! We haven't made any structs ourselves in this series yet, but we have used some. The `rl.Rectangle` we used the previous part is a struct.

Just above the line `main :: proc() {`, add this:

```C
Animation :: struct {
    texture: rl.Texture,
    current_frame: int,
    num_frames: int,
    frame_timer: f32,
    frame_length: f32,
}
```

This defines a new struct type name `Animation`. We can now use this type name `Animation` to create variables that hold all the information regard a specific animation. Each of those lines between the two {} are called fields. You can compare the names we chose of those field to those we already have inside `main`:

```C
player_run_texture := rl.LoadTexture("cat_run.png")
player_run_num_frames := 4
player_run_frame_timer: f32
player_run_current_frame: int
player_run_frame_length := f32(0.1)
```

I made the `Animation` struct based on these variables. What I aim to do is group all the things related to the `player_run` animation into a single `Animation` struct.

This means that we should now replace all those `player_run_...` variables with just a single Animation. Delete all those 5 lines and instead write:

```C
run_anim := Animation {
    texture = rl.LoadTexture("cat_run.png"),
    num_frames = 4,
    frame_length = 0.1,
}
```

This code makes a new variable called `run_anim` of type `Animation`. The lines between the `{}` are used to initialize the fields of the the animation. As you can see we are putting in the same values as `player_run_texture`, `player_run_num_frames` and `player_run_frame_length` had before.

Note that the struct field `frame_timer` corresponds to the old variable `player_run_frame_timer`. We do not need to mention that one, the Animation will contain it, but it should start at zero, and Odin default-initializes all fields to zero, unless we explicitly specify a value. Same thing for the field `current_frame`.

Since we are gonna add support for having multiple animations, it will be good to have a variable to hold the current animation. Just after the `run_anim`, add this:

```C
current_anim := run_anim
```

this just creates a new variable and copies the value of `run_anim` into it. Since `run_anim` is of type `Animation`, then so will `current_anim` be.

## Adding another animation

Currently the player's run animation is constantly playing, even when you stand still. This looks quite odd! Let's add a an animation for when the cat is standing still. Grab this image by right clicking it and choosing 'Save As'. Put it in the same folder as your odin code, next to the `cat_run.png` file we used in the previous part.

<img src="/odinraylib4/cat_idle.png" style="width: 100%; image-rendering: crisp-edges; image-rendering: pixelated; -ms-interpolation-mode: nearest-neighbor;" alt="a two-frame sprite sheet of a cat standing still">

As you can see from the image, this animation is just two frames. The cat just stands there but bobs up and down a little bit.


## That's it for today!