---
title: "Make games using Odin and Raylib #2: A controllable player"
date: 2024-02-11T15:47:11+02:00

cover:
  image: "/odinraylib2/cover.png"
---

In this second post about making games using Odin and Raylib we shall look at how to add a simple player character and make it moveable.

This post has a companion YouTube video that says pretty much the same stuff, but in video format. If you get confused by this post, then chances are that the companion video can help you understand. Here's the video:

Let's go!

## Drawing something more than just a blue background

The last post left off with our program looking like this:

```C
package game

import rl "vendor:raylib"

main :: proc() {
    rl.InitWindow(1280, 720, "My First Game")

    for !rl.WindowShouldClose() {
        rl.BeginDrawing()
        rl.ClearBackground(rl.BLUE)
        rl.EndDrawing()
    }

    rl.CloseWindow()
}
```

This game is currently just a pure blue background. Let's add something more to it. On the row after the line `rl.ClearBackground(rl.BLUE)`, add this:

`rl.DrawRectangleV({640,320}, {64, 64}, rl.GREEN)`

If you hit F7 and recompile you'll now see a green box on top of a blue background:

![The green box rendering on top of our blue background](/odinraylib2/green_box.png)

If we look at the line we just added we see that `rl.DrawRectangleV` seems to be a proc that accepts a couple of numbers and something at the end that tells it that we want a green box. The `{640, 320}` says that we want the box to be at the position `x = 640, y = 320`, where x and y are counted from the top left of the window. This is in the middle of our window (remember: we gave `rl.InitWindow` the window size `1280` by `720`). The `{64, 64}` says how big we want the box to be, i.e. we want it to be 64 pixels wide and 64 pixels high.

How can I know this proc exists in Raylib and also know what things it is possible to feed it? Try this: Open the Windows file explorer and go to `c:\odin\vendor`. In there you'll see a folder called `raylib`. The odin files in that folder are the ones that get imported by the line `import rl "vendor:raylib"` at the top of your program. Go inside the `raylib` folder and drag & drop `raylib.odin` into your code editor. Search for `DrawRectangleV` in there, you should find this line:

![DrawRectangleV in Raylib.odin](/odinraylib2/draw_rectangle.png)

This line tells us that `DrawRectangleV` accepts three _parameters_. The first one is the position of the rectangle, where we put in `{640, 320}`. The `Vector2` to the right of the word `position` tells us that this proc expects `position` to be of _type_ Vector2. What is Vector2? It's just a thing that consists of two decimal numbers. We can use a `Vector2` to denote positions and directions in 2 dimensional space. The second parameter `size` is also a `Vector2`, that's where we sent in `{64, 64}`.

Finally the last parameter is of type `Color`. Raylib comes with a few predefined colors, such as `rl.GREEN`. Try replacing `rl.GREEN` in our code with something like `{255, 180, 0, 255}`, which should make the box orange instead. Those four numbers denote red, green, blue and alpha respectively. `{255, 255, 255, 255}` means comletely white and `{0, 0, 0, 255}` means completely black. The alpha number at the end tells Raylib how transparent you want the rectangle to be.

![About colors in raylib](/odinraylib2/color.png)

> **SUBLIME TIP:** If you have both your `main.odin` file and `raylib.odin` open in Sublime you can also click on `DrawRectangleV` in your code and then press F12. Sublime will then try to find where this proc is defined in all your open files. It should jump to the correct line in `raylib.odin`. You can also press `Ctrl + Shift + R` and type `DrawRectangle` and get suggestions for all the procs in all your open files that contain those words.

> **NOTE:** There's no actual code inside `raylib.odin`, this file simply tells us what procs exist in Raylib and what parameters they need. This is because Raylib is written in the language C. The Odin code in `raylib.odin` is only there to instruct our Odin code on how it can talk to Raylib's C code.