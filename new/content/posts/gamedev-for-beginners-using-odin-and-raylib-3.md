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

Right click the image it in the same directory as `my_first_game.odin` and make sure the name is `cat_run.png`. On my computer it would thus be located at `c:\code\my_first_game\cat_run.png`.

As you see, this is a single image that contains four separate images of a cat. Each one of those images is called an animation frame. We will make our game display one of them at a time in order to to make the player look like a running cat.

Just below the line `player_grounded: bool`, add this line:
```C
player_run_texture := rl.LoadTexture("cat_run.png")
```

This `LoadTexture` proc tells Raylib to load the image you just downloaded and then send the image to your graphics card (GPU), so that it is ready for being drawn. When an image is loaded into the memory of the GPU, we usually call it a _texture_ instead of an _image_. We can use the variable `player_run_texture` later to actually draw the texture on the screen.

Let's replace the green rectangle with this cat texture. Replace the line `rl.DrawRectangleV(player_pos, {64, 64}, rl.GREEN)` with:

```C
rl.DrawTextureEx(player_run_texture, player_pos, 0, 4, rl.WHITE)
```

This proc tells raylib to draw the texture we loaded, and it will just like the `DrawRectangleV` proc draw it at the position `player_pos`. What are the other params? If we, just like we did in part 2, look inside `raylib.odin`, we can see that `DrawTextureEx` looks like this:

```C
DrawTextureEx :: proc(texture: Texture2D, position: Vector2, rotation: f32, scale: f32, tint: Color)
```

Comparing it to the line we just added, we see that we are supplying it with the value `0` for the rotation and the value `4` for the scale. A scale of `1` would mean the original size, while `4` means 4 times the size, or `400%`. We do that simply because the pixelart image we load is very tiny and need to be scale up somewhat. We supply the value `rl.WHITE` into `tint`. The `tint` color is a way to re-color the texture we are about to draw, and saying `rl.WHITE` here means that we will _not_ recolor it.

> **TECHNICAL GPU THING:** The way tint works is that your computer will multiply the colors of the pixels in the texture with the tint you supplied. Now, as you may remember from the previous post we supply Raylib colors in the format `{255, 255, 255, 255}` (this would make a completely white color). However, when the computer is actually drawing stuff using the GPU, then it usually uses colors that use decimal numbers that range from `0` to `1`. In this representation white would be `{1, 1, 1, 1}`. This is why tinting with white does not change the color of the texture when it is drawn: Because you're multiplying the red, green, blue and alpha of each pixel by `1`. The colors you supply to Raylib still use integer numbers between `0` and `255`, but those colors will be converted to the decimal variant for you.

If you run the game now, you'll see this:

![Drawing the cat_run.png texture instead of the green rectangle. Currently it displays all the four frames of our cat animation, but soon we'll it display just one at a time.](/odinraylib3/draw_texture.png "Drawing the cat_run.png texture instead of the green rectangle. Currently it displays all the four frames of our cat animation, but soon we'll it display just one at a time.")

The green rectangle sure has been replaced, but currently you see four cats! This is because we are not displaying one of the cat animation frames at a time yet. Currently we are drawing the whole image.

Before we continue, you might want to change you background color, that blue is a bit jarring compared to the color of the cat. Change this line `rl.ClearBackground(rl.BLUE)` and replace `rl.BLUE` with `{110, 184, 168, 255}` in order to give your game a calmer blue background.

## Displaying just one of the four frames in the animation

Currently our game is drawing the whole `cat_run.png` image, which contains four frames, but we want to only draw one of those frames at a time. Let's start by making it display only the first frame, and then later we'll worry about making it switch to the other frames.

![](/odinraylib3/draw_first_square.png "")

This section will be the most code we've so far written in one go, so please read this section several times or watch the video version of this post if you ge confused.

We know that our animation contains four frames, let's put that knowledge in a variable we can use, under the line `player_run_texture := rl.LoadTexture("cat_run.png")`, add this line:
```C
player_run_num_frames := 4
```
Now we will replace the line `rl.DrawTextureEx(player_run_texture, player_pos, 0, 4, rl.WHITE)` with this whole bunch of code (don't worry, I'll go through it in detail):

```C
player_run_width := f32(player_run_texture.width)
player_run_height := f32(player_run_texture.height)

draw_player_source := rl.Rectangle {
    x = 0,
    y = 0,
    width = player_run_width / f32(player_run_num_frames),
    height = player_run_height,
}

draw_player_dest := rl.Rectangle {
    x = player_pos.x,
    y = player_pos.y,
    width = player_run_width * 4 / f32(player_run_num_frames),
    height = player_run_height * 4
}

rl.DrawTexturePro(player_run_texture, draw_player_source, draw_player_dest, 0, 0, rl.WHITE)
```

The first two lines, `player_run_width := f32(player_run_texture.width)` and `player_run_height := f32(player_run_texture.height)` are there to just make the rest of the code easier to read. We are fetching the width and the height of the `player_run_texture`. Here you see that we've once surrounded things with `f32()`, this is because the texture stores the width and height as integer numbers, but we need them to be decimal numbers later, so we do this conversion at once here.

The size of the `cat_run.png` image is 72 pixels wide and 18 pixels high, which is what those numbers `player_run_width` and `player_run_height` will contain, respectively. In fact, 72 divided by 4 is 18, which means we have 4 frames that are each 18 by 18 pixels big.

![](/odinraylib3/18pixels.png "")

Now, in order to make our game only show the part that that is the first 18x18 pixels of the `cat_run.png` image, we need to create a _rectangle_ that represents this. These lines creates such a rectangle:

```C
draw_player_source := rl.Rectangle {
    x = 0,
    y = 0,
    width = player_run_width / f32(player_run_num_frames),
    height = player_run_height,
}
```

A rectangle is a thing in Raylib that has a position (x, y) and a size (width, height). In this case we say that the position is `x = 0` and `y = 0`. This position is measured _inside_ the `cat_run.png` image, it just means the upper left corner of that image.

The `width = player_run_width / f32(player_run_num_frames)` part means that this rectangle will be `72 / 4 = 18` pixels wide. We could of course have written just 18 and not used the the variables `player_run_width` and `play_run_num_frames` here, but by doing it this way we could modify `cat_run.png` to become wider without our game looking weird.

The height of the rectangle is simply `player_run_height`, in this case 18 pixels. This matches up with what we expect from the image above.

The rectangle is named `draw_player_source`, in this case, _source_ refers to "what portion of the texture do you wish to draw?"

Now we must also know where on the screen to draw our player. Previously we have just used the player's position, but in this case we do things a bit differently. We make a rectangle called `draw_player_dest`, where `dest` is short for _destination_. It is the rectangle _on the screen_ within which we want to draw:

```C
draw_player_dest := rl.Rectangle {
    x = player_pos.x,
    y = player_pos.y,
    width = player_run_width * 4 / f32(player_run_num_frames),
    height = player_run_height * 4
}
```

![](/odinraylib3/dest_rectangle.png "")

In this case the rectangle will have the position `x = player_pos.x`, `y = player_pos.y`, so that it sits at the player's position. The width of this rectangle we have written as `width = player_run_width * 4 / f32(player_run_num_frames)`, which is similar to the `source` rectangle, but note the `* 4`. We add this so that the dest rectangle gets a bit scaled, simply because it gets so tiny its hard to see otherwise. Same thing for the height: `height = player_run_height * 4`, we have added a `* 4` here again in order to scale the image up. This means that the width and height will be 72x72 pixels.

Finally, we will use these two rectangles and our texture to draw it on the screen. The final line of the code we wrote is this:

```C
rl.DrawTexturePro(player_run_texture, draw_player_source, draw_player_dest, 0, 0, rl.WHITE)
```

As usual, you could look into `raylib.odin` to see what the parameters of `DrawTexturePro actually are:

```C
DrawTexturePro :: proc(texture: Texture2D, source, dest: Rectangle, origin: Vector2, rotation: f32, tint: Color)
```

With this proc `DrawTexturePro` we can tell Raylib to draw the texture `player_run_texture`, but only draw the parts of the texture that `draw_player_source` says, in this case the first 18x18 pixels of the texture. It will use `draw_player_dest` to choose what portion of the screen to draw onto, which will be a 72x72 pixels area of the screen located at the position of the player. 

We are feeding `0` into `rotation`, meaning that the texture will be drawn without rotation. The `origin` is also fed a zero. You may notice that `origin` is actually of type `Vector2`, which contains _two_ decimal numbers. In Odin you can write just `0` to zero both x and the y part of a `Vector2`. We'll return to the origin later, but for now I'll just say that it is a offset relative to the `dest` rectangle, making it possible to shift the position of the texture. `tint` is just like we've seen previously.

Now, if you run the game you'll see this:

<figure>
<video autoplay loop muted width="100%"><source src="/odinraylib3/one_frame.mp4"></video>
<figcaption>meow</figcaption>
</figure>

Now we are just displaying the first frame of the player! Now we just need to make the our code switch animation frame every now and then, and then our cat will run!

## Making the animation switch frame

