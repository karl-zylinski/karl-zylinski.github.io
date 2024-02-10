---
title: "Make games with Odin and Raylib #1: Setup + First Program"
date: 2024-02-09T15:47:11+02:00

draft: true

cover:
  image: "/odinraylib1/odinwhat.png"
---

Hello, welcome to the first post in this blog series on making games with Odin and Raylib. The target audience for this series is people have never done video games programming, but who have perhaps done a tiny bit of programming before. Perhaps you know what a variable and a function is and maybe you've also come across structs/classes before. Maybe you've seen for/while loops in other languages. If that sounds like you, then I urge you to give all this a go, you'll might find it fun!

This blog has a companion YouTube video that says pretty much the same stuff, but in video format: LINK

I'll begin by describing what Odin and Raylib is, after that we'll proceed to download the Odin compiler, required software, setup a code editor and finally we'll make our first tiny Odin + Raylib program.

I work on Windows, so some instructions might be a bit Windows-centric, but it should be mostly the same if you are on mac or Linux.

## What's Odin?
![PortableBuildTools](/odinraylib1/odinwhat.png)

Odin, sometimes referred to as Odinlang, is a programming language that tries to be a modern alternative to the old language C. It is also a fairly simple language, making it beginner friendly. Odin is a so-called 'low level' language, meaning that you have a great deal of control, making it suitable for creating video games. Why do video games need so much control? Because they need often need to do a lot of things very quickly, so there is great benefit in being able to tell the computer what to do in great detail.

If you come from a language such as Javascript, C# or Python, then please don't be discouraged by this, because Odin is, as I usually say, "low level with a high level feeling". You have lots of control, but there are also many comfortable modern features. All this, while still keeping the language very simple.

## What's Raylib?

Raylib is a library, meaning a collection of code, for drawing graphics, creating windows, processing user input such as key and gamepad button presses and also outputting sound. It's a fantastic way to learn video game programming.

Note that Raylib is not a game engine. What we will do here is write Odin code that uses Raylib and turn that code into an executable game that you can run. It is a straight-to-the-point and fun way to make games.

## Can you make proper games with Odin and Raylib?

Yes. I've recently made and shipped a whole game called CAT & ONION using Odin and Raylib. Here's a trailer, to give you a sense of what's possible:

{{< youtube gggtSXiJkKc >}}

## Download and setup the Odin compiler

Before we get started with the code we'll have to download and setup some software. I urge you to push on through these perhaps boring setup steps, as things will get much more fun once we start writing code.

In order to turn the Odin code we are about to write into a program you can run, you need to download the Odin compiler. Compilation is the process by which the computer takes code and outputs a program you can run. While you can follow the instructions at https://odin-lang.org/docs/install/, I would instead recommend this simpler approach:

> NOTE: If you are on mac or Linux, just follow the official instructions, my simplified instructions are for Windows only.

Firstly, download the compiler from here: https://github.com/odin-lang/Odin/releases/latest. Scroll to the bottom and download the Odin compiler for the platform you are on. If you are on Windows it will be a file called `
odin-windows-amd64-dev-<year>-<month>.zip`. Download the file and extract the contents to the folder `c:\odin`.

Secondly, the official Odin install instructions says to install Visual Studio, which contains some additional software and the Windows SDK, which Odin needs in order to compile programs. However, I propose that you instead download and install 'PortableBuildTools'. It is a package that contains the software and Windows SDK that the Odin compiler needs, without having to install the giant Visual Studio code editor that you'll probably don't need anyways. Go here https://github.com/Data-Oriented-House/PortableBuildTools and download the latest release. Run the program you downloaded (you might get a windows security warning because the program isn't from a certified developer, in that case you have to click 'more info' and 'run anyway'). When you run it you'll see this:

![PortableBuildTools](/odinraylib1/portable_build_tools.png)

Click the 'Add to Environment' checkbox, so that the Odin compiler can find the installed software. Thereafter you need to accept Microsoft's license agreement and after that click Install. When it's all done the installer might tell you to log out and in again, in which case you should do that.

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
When you've typed it all in, go ahead and save it to `c:\code\my_first_game\my_first_game.odin` (you'll have to create those code and my_first_game directories).

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

and then save it. Name it something like `my_first_game.sublime-build`. Make sure to save it in the folder Sublime suggests.

> NOTE: We use the absolute c:/odin/odin.exe path. You could also add c:/odin to the PATH environment variable. But it doesn't really matter. Also please note that I use / here instead of \\. If you use \\ then be aware that you have to type two of them, i.e. \\\\.

The `working_dir` denotes the working directory, meaning the directory from which we will run the compiler. The `shell_cmd` part is the command that our build system will run. In this case it will tell the odin compiler to compile and run our code. The period `.` is an old-school way to denote 'the current directory', which means whatever we wrote on the `working_dir` line. Odin treats all files inside a directory as part of a 'package', so it will take all the code files inside the working directory and compile them as a package. However, we only have one file (`main.odin`), but if we add more files later then they will get picked up as well.

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

Let's start by looking at the line `main :: proc() {`. This how you begin a function in Odin, or as they are usually called in Odin, a procedure. The contents of the procedure (proc) goes between the two curly braces (`{}`). The name of the proc is in this case `main`. It is in the `main` proc that Odin programs start. When the `main` proc finishes, then your program will shut down.

As you see there are a bunch of lines that all start with `rl`. These are all calls to Raylib, i.e. code that is instructing Raylib to do stuff. As mentioned before, Raylib can draw graphics and create windows. The line `rl.InitWindow(1280, 720, "My First Game")` creates a window of size 1280x720 pixels and gives it the window a title.

The line `for !rl.WindowShouldClose() {` makes a loop that runs for as long as the window created by Raylib wants to stay open. That is, until you press the close button on the window. The stuff between the two curly braces `{}` is the code that the loop will run. You may have seen `while` and `for` loops before. In Odin there are only `for` loops. If you make a `for` loop in Odin that has only one condition, such as asking Raylib if the Window should stay open, then it works the same way as `while` loops in other languages.

This loop will be the 'main loop' of our video game. Video games consist of constantly updating images on the screen, called frames. Gamers often talk about how many 'FPS' or Frames Per Second their game runs at, which essentially means how many times per second our `for` manages to loop. The more stuff you add in there, the lower the FPS.

`rl.BeginDrawing()` instructs Raylib to start a new frame and `rl.EndDrawing()` instructs Raylib to end the frame. In-between those two lines there's a line that clears the whole screen with a color, which is what makes our game look blue. 

> NOTE: You can't skip the Begin/End calls. Internally, the `rl.EndDrawing()` call does lots of stuff like also check if the close button was called and make sure that the window is moveable. It is also responsible for asking Windows for if any keys or gamepad button were pressed. It is also in there that it actually sends off the things we want to draw to the graphics card.

Our loop will terminate if the user presses the close button on the window, in which case it will leave the loop and then run the final line of `main`: `rl.CloseWindow()`, after that our game shuts down because the `main` proc is done.

Finally, I'll just say something about the first two lines of our program:
`import rl "vendor:raylib"` tells the Odin compiler that we want to use Raylib in our program. The Odin compiler comes with built in support for Raylib, as part of the "vendor" collection of libraries. There are numerous other vendor libraries, you can look at the other ones by going to `c:\odin\vendor`. The `rl` just after `import` tells the compiler to give raylib the alias `rl`, if we instead had written `import "vendor:raylib"`, then we would have had to write `raylib.InitWindow(...` etc everywhere, which would have gotten a bit cumbersome.

What about the first line `package game`? All files within a folder in Odin are said to be part of a package. And all those files must start with the same `package NAME` line. The name you put there is a technicality only used if you export your code as library, for an executable like our game it does not matter, so we'll just put `game` there.

## Did anything go wrong?

If your program did not compile, chances are you mistyped something in the code. When you compile and there is an error, you'll see information in the console at the bottom of your sublime window:

![Compilation Error](/odinraylib1/error.png)

As you can see in this image, I accidentally typed `BegunDrawing` instead of `BeginDrawing`. The error message is in this case helpful and suggests what I might have meant instead.

## That's it!

I know it might have felt a bit overwhelming with the compiler setup and all that, but from here on we'll focus on just writing code and talking about how games work.

I hope to see you in part 2, where we will write some _gameplay code_.