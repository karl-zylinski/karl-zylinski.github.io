---
title: "Make games using Odin and Raylib #1: Setup + First Program"
date: 2024-02-09T15:47:11+02:00

draft: false

cover:
  image: "/odinraylib1/cover.png"
---

Hello, welcome to the first post in this blog series on making games with Odin and Raylib. The target audience for this series is people who have never done video games programming, but who have perhaps done a tiny bit of other programming before.

This blog has a companion YouTube video that says pretty much the same stuff, but in video format: LINK

I'll begin by describing what Odin and Raylib is, after that we'll download the Odin compiler, required software, setup a code editor and finally we'll make our first tiny Odin + Raylib program.

I work on Windows, so some instructions might be a bit Windows-centric, but I will try to put in notes for what you'll need to do on mac or Linux.

## What's Odin?
![Odin Logo wish Question Mark](/odinraylib1/odinwhat.png)

Odin, sometimes referred to as Odinlang, is a programming language that tries to be a modern alternative to the old language C. It is also a fairly simple language, making it beginner friendly. Odin is a so-called 'low level' language, meaning that you have a great deal of control, making it suitable for creating video games. Why do video games need so much control? Because they need often need to do a lot of things very quickly, so there is great benefit in being able to tell the computer what to do in great detail.

If you come from a language such as Javascript, C# or Python, then please don't be discouraged by this, because Odin is, as I usually say, "low level with a high level feeling". You have lots of control, but there are also many comfortable modern features. All this, while still keeping the language very simple.

## What's Raylib?

Raylib is a library, meaning a collection of code, for drawing graphics, creating windows, processing user input such as key and gamepad button presses and also outputting sound. It's a fantastic way to learn video game programming.

Note that Raylib is not a game engine. What we will do here is write Odin code that uses Raylib and turn that code into an executable game that you can run. It is a straight-to-the-point and fun way to make games.

Note that Odin comes with build in support for Raylib, you do not need to install anything extra after we've gotten Odin up and running. You can read more about Raylib on it's website: https://www.raylib.com/.  

## Can you make proper games with Odin and Raylib?

Yes. I've recently made and shipped a whole game called CAT & ONION using Odin and Raylib. Here's a trailer, to give you a sense of what's possible:

{{< youtube gggtSXiJkKc >}}

## Download and setup the Odin compiler

Before we get started with the code we'll have to download and setup some software. I urge you to push on through these perhaps boring setup steps, as things will get much more fun once we start writing code.

In order to turn the Odin code we are about to write into a program you can run, you need to download the Odin compiler. Compilation is the process by which the computer takes code and outputs a program you can run. While you can follow the instructions at https://odin-lang.org/docs/install/, I would instead recommend this simpler approach:

> **Mac / Linux:** If you are on mac or Linux, just follow the official instructions and skip to the next section. My simplified instructions are for Windows only.

Firstly, download the compiler from here: https://github.com/odin-lang/Odin/releases/latest. Scroll to the bottom and download the Odin compiler for the platform you are on. If you are on Windows it will be a file called `
odin-windows-amd64-dev-<year>-<month>.zip`. Download the file and extract the contents to the folder `c:\odin`.

Secondly, the official Odin install instructions says to install Visual Studio, which contains some additional software and the Windows SDK, which Odin needs in order to compile programs. However, I propose that you instead download and install 'PortableBuildTools'. It is a package that contains the software and Windows SDK that the Odin compiler needs, without having to install the giant Visual Studio code editor that you'll probably don't need anyways. Go here https://github.com/Data-Oriented-House/PortableBuildTools and download the latest release. Run the program you downloaded (you might get a windows security warning because the program isn't from a certified developer, in that case you have to click 'more info' and 'run anyway'). When you run it you'll see this:

![PortableBuildTools](/odinraylib1/portable_build_tools.png)

Click the 'Add to Environment' checkbox, so that the Odin compiler can find the installed software. Thereafter you need to accept a license agreement and then click Install. When it's all done the installer might tell you to log out and in again, in which case you should do that.

You now have a working Odin compiler.

## Download Sublime Text, a code editor

Soon we'll write some Odin code. But first we need a program to write our code in!

I write Odin code using Sublime Text. It is a simple and snappy code editor. Download and install it from here: https://www.sublimetext.com/

## Let's write some Odin code!

Open Sublime, type in the code below! Let's first make it compile and run and then I'll go through what it all means.

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
When you've typed it all in, go ahead and save it to `c:\code\my_first_game\my_first_game.odin` (you'll have to create those `code` and `my_first_game` directories).

## Let's compile the code!

In order to compile the code from within Sublime Text, we'll need to create a Build System. A Build System consists of instructions for Sublime Text that tells it what command to run in order to compile our code. In Sublime do this:

![SublimeText](/odinraylib1/sublime.png)

Click the menu button and go to Tools -> Build System -> New Build System...

Sublime will pop up a new build system file, in it type this:
```
{
    "working_dir": "c:/code/my_first_game",
    "shell_cmd": "c:/odin/odin.exe run ."
}
```

and then save it. Name it something like `my_first_game.sublime-build`. Make sure to save it in the folder Sublime suggests. Make sure you use the `.sublime-build` ending, otherwise Sublime will not find it.

The `working_dir` denotes the working directory, meaning the directory from which we will run the compiler. We let this point to the directory where we put our code. The `shell_cmd` part is the command that our build system will run. In this case it will tell the odin compiler to compile and run our code. The period `.` in `c:/odin/odin.exe run .` is an old-school way to denote 'the current directory', which means whatever we wrote on the `working_dir` line. Odin treats all files inside a directory as part of a 'package', so it will take all the code files inside the working directory and compile them as a package. However, we only have one file (`main.odin`), but if we add more odin files later then they will get picked up as well.

> **NOTE:** We use the absolute `c:/odin/odin.exe` path. You could also add c:/odin to the PATH environment variable in order to make `odin.exe` available from anywhere on the system. But it doesn't really matter. Also please note that I use `/` in the path, instead of `\`. If you use `\` then be aware that you have to type two of them, i.e. `\\`.

> **Mac / Linux:** A good place to put the code is `~/code/my_first_game`, where `~` denotes your home directory. The odin compiler you can put at `~/odin`, or where ever you like really.

Return to `main.odin` and go to the menu -> Tools -> Build System -> my_first_game. If you now press F7 or Ctrl+B, then Sublime will use your shiny new build system. It should compile and run your first Odin program:

![SublimeText](/odinraylib1/my_first_game.png)

You should now see a window with a blue background!

> NOTE: `odin run .` will compile and run your game. You can also have the build system execute `odin build .` in which case it does not execute your game after a successful compilation. In both cases `my_first_game.exe` is outputted next to your odin code, so you can run the game later without needing to involve the odin compiler.

## So what does that code we typed in really do?

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

Let's start by looking at the line `main :: proc() {`. This line defines a new function called `main`. In Odin, functions are usually named procedures (shortened proc), which we will call them from now on. Procs contains code and can in turn call other procs, which makes it possible combine procs to write however big a game you may want. When an Odin program starts, it looks for a proc called `main` and starts the game from the top of that proc. The proc runs from the opening curly brace `{`, to the matching closing curly brace `}`, as shown below:

![Main Proc with arrows of where it starts and ends](/odinraylib1/main.png)

When the closing curly brace `}` of the main proc is reached, then the game has finished running and will shut down. In essence your whole game goes in-between those two curly braces of the main proc.

Looking inside the `main` proc we see a bunch of lines that all start with `rl`. These are all calls to Raylib, i.e. code that is instructing Raylib to do stuff. As mentioned before, Raylib can draw graphics and create windows. The line `rl.InitWindow(1280, 720, "My First Game")` creates a window of size 1280x720 pixels and gives the window a title.

> **NOTE:** You do not need to use `;` at the end of the lines, as you might do in other languages.

The line `for !rl.WindowShouldClose() {` makes a loop that runs for as long as the window created by Raylib wants to stay open. `rl.WindowShouldClose()` is a proc in Raylib that essentially tells us if the user pressed the close button of the window. But we want to know if the user _didn't_ press it, which is why we put the `!` in front to negate the answer from raylib. The stuff between the two curly braces `{}` is the code that the loop will run.

![Image saying that the code inbetween the two curly braces of the for loop will run over and over as long as the window isn't closed](/odinraylib1/main_loop.png)


This loop will be the 'main loop' of our video game. Video games consist of constantly updating images on the screen, called frames. Gamers often talk about how many 'FPS' or Frames Per Second their game runs at, which essentially means how many times per second our `for` manages to loop. The more stuff you add in there, the lower the FPS.

> **NOTE:** You may have seen `while` and `for` loops before. In Odin there are only `for` loops. If you make a `for` loop in Odin that has only one condition, such as asking Raylib if the Window should stay open, then it works the same way as `while` loops in other languages.

`rl.BeginDrawing()` instructs Raylib to start a new frame and `rl.EndDrawing()` instructs Raylib to end the frame. In-between those two lines there's a line that clears the whole screen with a color, which is what makes our game look blue.

> **WARNING:** You can't skip the Begin/End calls. Internally, the `rl.EndDrawing()` call does lots of stuff like also check if the close button was called and make sure that the window is moveable. It is also responsible for asking Windows for if any keys or gamepad button were pressed. It is also in there that it actually sends off the things we want to draw to the graphics card.

Our loop will terminate if the user presses the close button on the window, in which case it will leave the loop and then run the final line of `main`: `rl.CloseWindow()`, after that our game shuts down because the `main` proc is done.

Finally, I'll just say something about the first two lines of our program:
`import rl "vendor:raylib"` tells the Odin compiler that we want to use Raylib in our program. The Odin compiler comes with built in support for Raylib, as part of the "vendor" collection of libraries. We say that we are importing the Raylib _package_, anything you import in this way will end up under a name such as `rl.`, and the things you can access by typing `rl.` is said to be part of the same package.
The `rl` just after `import` tells the compiler to give raylib the alias `rl`, if we instead had written `import "vendor:raylib"`, then we would have had to write `raylib.InitWindow(...` etc everywhere, which would have gotten a bit cumbersome.

> **NOTE:** There are numerous other vendor packages, you can see them all by looking into `c:\odin\vendor`. You'll find stuff like DirectX, OpenGL and SDL in there.

What about the first line `package game`? All files within a folder in Odin are said to be part of a package. And all those files must start with the same `package NAME` line. The name you put there is a technicality only used if you export your code as library, for an executable like our game it does not matter, so we'll just put `game` there.

## Did anything go wrong?

If your program did not compile, chances are you mistyped something in the code. When you compile and there is an error, you'll see information in the console at the bottom of your sublime window:

![Compilation Error](/odinraylib1/error.png)

As you can see in this image, I accidentally typed `BegunDrawing` instead of `BeginDrawing`. The error message is in this case helpful and suggests what I might have meant instead.

## That's it!

I hope to see you in part 2, where we will write some _gameplay code_.