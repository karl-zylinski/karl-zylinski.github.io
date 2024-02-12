---
title: "Make games using Odin and Raylib #1: Setup + First Program"
date: 2024-02-09T15:47:11+02:00

draft: true

cover:
  image: "/odinraylib1/cover.png"
---

**Hello and welcome to the first post in this blog series on making games with Odin and Raylib**. Throughout this series we will create a small 2D game with simple platforming mechanics. When we are done you'll be able to extend the game, or take the knowledge you've learnt and make other, more complex games!

**This series is aimed at people with little to no programming experience**. Being a bit technical and having an interestest in games will help. Also, people who do know programming but have never done video games programming may find this interesting.

This post has a **companion YouTube video** that says pretty much the same stuff:
{{<youtube PUTIDHERE>}}
If you get confused by this post, then chances are that the companion video can help you understand.

I'll begin by describing what Odin and Raylib is, after that we'll download the Odin compiler, download some required software, setup a code editor and finally we'll make our first tiny Odin + Raylib program.

I use Windows, so some instructions might be a bit Windows-centric. I will try to put in notes for what you need to do on mac and Linux.

## What's Odin?
![Odin Logo wish Question Mark](/odinraylib1/odinwhat.png)

[Odin](https://www.odin-lang.org), sometimes referred to as Odinlang, is a programming language that tries to be a modern alternative to the old language C. It is a fairly simple language, making it beginner friendly. Odin is a so-called 'low level' language, meaning that you have a great deal of control, making it suitable for creating video games. Why do video games need so much control? Because they often need to do a lot of things very quickly, making sure there are no hitches and jerks, so there is benefit in being able to tell the computer what to do in great detail.

The term _low level_ refers to the programming language being "close to the hardware", which in essence means that there are few layers of automatic stuff between the code and the computer's processor. The opposite term _high level_ is often used to describe languages that does a great deal for you automatically, at the cost of having less control.

If you come from a high level language such as Javascript, C# or Python, then please don't be discouraged by all this low levelness and control. Because Odin is, as I usually say, "low level with a high level feeling". You have lots of control, but there are also many comfortable modern features to help you write expressive code. All this, while still keeping the language very simple.

## What's Raylib?

[Raylib](https://www.raylib.com/) is a library, meaning a collection of ready-made code. It lets you draw graphics, create windows, process keyboard, mouse and gamepad input, and also output sound. It's a simple and easy-to-use library created for learning game programming, so it's a great fit for your first game.

<figure>
{{<youtube -RROryQQq-s >}}
<figcaption>A showcase of things people made with Raylib</figcaption>
</figure>


Note that Raylib is not a game engine. What we will do here is write Odin code that uses Raylib and from that code output a stand-alone executable game that you can run. It is a straight-to-the-point and fun way to make games.

Odin comes with built in support for Raylib, you do not need to install anything extra after we've gotten Odin up and running.

## Can you make proper games with Odin and Raylib?

Yes. I've recently made and shipped a whole game called CAT & ONION using Odin and Raylib. Here's a trailer, to give you a sense of what's possible:

<figure>
{{< youtube gggtSXiJkKc >}}
<figcaption>Trailer for my game CAT & ONION. The game is made completely using Odin and Raylib!</figcaption>
</figure>

## Download and setup the Odin compiler

Before we get started with the code we'll have to download and setup some software. I urge you to push on through these perhaps boring setup steps, as things will get much more fun once we start writing code.

In order to turn the Odin code we are about to write into a program you can run, you need to download the Odin _compiler_. Compilation is the process by which the computer takes code and outputs a program you can run. While you can follow the official instructions at https://odin-lang.org/docs/install/, I would instead recommend this simpler approach:

> **MAC / LINUX:** If you are on mac or Linux, just follow the official instructions and skip to the next section. My simplified instructions are for Windows only.

Firstly, download the compiler from here: https://github.com/odin-lang/Odin/releases/latest. Scroll to the bottom and download the file called `
odin-windows-amd64-dev-<year>-<month>.zip`. When the download has finished, extract the contents to the folder `c:\odin`. You should end up with something that looks like this:

![Odin unzipped into c:\\odin](/odinraylib1/odin_folder.png "Odin compiler unzipped into c:\odin")

Secondly, the official Odin install instructions says to install Visual Studio, which contains some additional software and the Windows SDK, which Odin needs in order to compile Windows programs. However, I propose that you instead download and install 'PortableBuildTools'. It is a tool that downloads those things that the Odin compiler needs, without having to install the giant Visual Studio code editor that you'll probably don't need anyways. Go here https://github.com/Data-Oriented-House/PortableBuildTools and download the latest release. Run the program you downloaded. 

> **NOTE:** You might get a windows security warning because PortableBuildTools isn't from a certified developer, in that case you have to click 'more info' and 'run anyway'. It is safe application used by many, so nothing to worry about.

When you run it you'll see this:

![PortableBuildTools](/odinraylib1/portable_build_tools.png "Install the additional software the Odin compiler needs by clicking 1, 2 and then 3")

Click the 'Add to Environment' checkbox, so that the Odin compiler can find the installed software. Thereafter you need to accept a license agreement and then click Install. When it's all done the installer might tell you to log out and in again, in which case you should do that.

You now have a working Odin compiler.

## Download Sublime Text, a code editor

Soon we'll write some Odin code. But first we need a program to write our code in!

I write Odin code using Sublime Text. It is a simple and snappy code editor. Download and install it from here: https://www.sublimetext.com/

Throughout this series I'll try to sneak in some good Sublime tips every now and then, to accelerate your workflow.

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
When you've typed it all in, go ahead and save it to `c:\code\my_first_game\my_first_game.odin`. You'll have to create those `code` and `my_first_game` directories.

## Let's compile the code and run our game!

In order to compile the code from within Sublime Text, we'll need to create a Build System.

> **IF YOU CHOSE TO NOT USE SUBLIME TEXT:** This section won't make much sense. In that case you can compile the game by opening the windows command-line `cmd` and go to the `c:\code\my_first_game` directory by typing `cd \code\my_first_game`  and from there compile and run the game by typing `c:\odin\odin run .`

A Build System is small file that instructs Sublime how to compile our game. We do this now so you can quickly build and run your game from within sublime for the rest of this series. In Sublime, click the menu button and go to Tools -> Build System -> New Build System...

![Adding a new Build System in Sublime Text](/odinraylib1/sublime.png "Adding a new build system in Sublime Text: Press the menu button in the top left corner and go to Tools -> Build System -> New Build System...")

Sublime will pop up a new build system file, in it type this:
```
{
    "working_dir": "c:/code/my_first_game",
    "shell_cmd": "c:/odin/odin run ."
}
```

and then save it. Name it something like `my_first_game.sublime-build`. Make sure to save it in the folder Sublime suggests (on Windows it probably wants you to save it in `%appdata%\Sublime Text 3\Packages\User`). Make sure you use the `.sublime-build` ending, otherwise Sublime will not find it.

This file tells Sublime to compile our game using the command `c:/odin/odin run .` (note the period at the end). The `working_dir` part tells sublime to run the command from within the directory where we saved our code. You do not need to tell the Odin compiler exactly which files we wish to compile, the period `.` at the end of the command denotes the 'current directory', which means that it takes all the files in the `working_dir` and compiles them into a package. However, we only have one file (`main.odin`), but if we add more odin files later then they will get picked up as well.

> **NOTE:** We use the absolute `c:/odin/odin` path. You could also add `c:/odin` to the PATH environment variable in order to make `odin` available from anywhere on the system. But it doesn't really matter. Also please note that I use `/` in the path, instead of `\`. If you use `\` then be aware that you have to type two of them, i.e. `\\`. This is due to `\` being a special character.

> **MAC / LINUX:** A good place to put the code is `~/code/my_first_game`, where `~` denotes your home directory. The odin compiler you can put at `~/odin`, or where ever you like really. If you installed Odin using a package manager then it might be available system-wide by just typing `odin` in a terminal.

Return to `main.odin` and go to the top left menu -> Tools -> Build System -> my_first_game. If you now press F7 or Ctrl+B, then Sublime will use your shiny new build system to build and run your game. It should compile and run your first Odin program:

![SublimeText](/odinraylib1/my_first_game.png "Our first Odin + Raylib program: It's just a blue background for now, but soon we'll have gameplay in there!")

You should now see a window with a blue background! It's not much, but soon we'll add things moving around in there!

> **NOTE:** `odin run .` will compile and run your game. You can also have the build system execute `odin build .` in which case it does not execute your game after a successful compilation. In both cases `my_first_game.exe` is outputted next to your odin code, so you can run the game later without needing to involve the odin compiler.

## Did anything go wrong?

If your program did not compile, chances are you mistyped something in the code. When you compile and there is an error, you'll see information in the console at the bottom of your sublime window:

![Compilation Error](/odinraylib1/error.png "If you mistyped anything, then the errors from the compilation will be visible in the console")

As you can see in this image, I accidentally typed `BegunDrawing` instead of `BeginDrawing`. The error message is in this case helpful and suggests what I might have meant instead.

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

Let's start by looking at the line `main :: proc() {`. This line defines a new _procedure_. A procedure, or proc as I will call it, is a block of code with a name attached. The code inside the proc can be run if you know the proc's name. In this case we are creating a proc called `main`. When an Odin program starts, it looks for a proc called `main` and starts the program from the top of that proc. The proc runs from the opening curly brace `{`, to the matching closing curly brace `}`, as shown below:

![Main Proc with arrows of where it starts and ends](/odinraylib1/main.png "The program will start at the top of the `main` proc. All procs run from the opening curly brace `{` to the closing curly brace `}`")

When the closing curly brace `}` of the main proc is reached, then the game has finished running and will shut down. In essence your whole game goes in-between those two curly braces of the main proc.

> **INTERESTING ODIN THING:** Those who have programmed before might recognize procedures by the word function. Those two words mostly mean the same thing, but the word procedure has a bit of a wider defintion.

Looking inside the `main` proc we see a bunch of lines that all start with `rl`. All those lines are _calls_ to procs within Raylib, i.e. code that is instructing Raylib to do stuff. By _proc call_ I mean that you use the name of a proc in order to run the code inside that proc. As mentioned before, Raylib can draw graphics, create windows and much more. The line `rl.InitWindow(1280, 720, "My First Game")` creates a window of size 1280x720 pixels and gives the window the title "My First Game".

The line `for !rl.WindowShouldClose() {` makes a loop that runs for as long as the window created by Raylib wants to stay open. `rl.WindowShouldClose()` is a proc in Raylib that essentially tells us if the user pressed the close button of the window. But we want to know if the user _didn't_ press it, which is why we put the `!` in front to negate the answer from Raylib. The stuff between the two curly braces `{}` is the code that the loop will run, over and over again, until the user tries to close the window.
![Image saying that the code inbetween the two curly braces of the for loop will run over and over as long as the window isn't closed](/odinraylib1/main_loop.png "This loop will run as long as the window wants to stay open. The code between the curly braces will run over and over again.")

This loop will be the 'main loop' of our video game. Video games consist of constantly updating images on the screen, called frames. Gamers often talk about how many 'FPS' or Frames Per Second their game runs at, which essentially means how many times per second our `for` manages to loop. The more stuff you add in there, the lower the FPS.

> **NOTE:** You may have seen `while` and `for` loops before. In Odin there are only `for` loops. If you make a `for` loop in Odin that has only one condition, such as asking Raylib if the Window should stay open, then it works the same way as `while` loops in other languages.

The line `rl.BeginDrawing()` instructs Raylib to start a new frame and `rl.EndDrawing()` instructs Raylib to end the frame. In-between those two lines there's a line that clears the whole screen with a color, which is what makes our game look blue.

> **WARNING:** You can't skip the BeginDrawing/EndDrawing calls. Internally, the `rl.EndDrawing()` proc does lots of housekeeping stuff such as making it possible to move and close the window. It is also responsible for asking Windows if any keys or gamepad button were pressed. It is also in there that it actually sends off the things we want to draw to the graphics card.

Our loop will terminate if the user presses the close button on the window, in which case it will leave the loop and then run the final line of `main`: `rl.CloseWindow()`, after that our game shuts down because the `main` proc is done.

Finally, I'll just say something about the first two lines of our program:
`import rl "vendor:raylib"` tells the Odin compiler that we want to use Raylib in our program. The Odin compiler comes with built in support for Raylib, as part of the "vendor" collection of libraries. We say that we are importing the Raylib _package_. All those things in Raylib will end up under the `rl` prefix, which is why those procs we used in our program all started with `rl`

> **INTERESTING ODIN THING:** There are numerous other vendor packages, you can see them all by looking into `c:\odin\vendor`. You'll find stuff like DirectX, OpenGL and SDL in there.

What about the first line: `package game`? All odin files within a directory are said to be part of the same package. And all those files must start with the same `package NAME` line. The name you put there is a technicality only used if you export your code as library, for an executable like our game it does not matter, so we'll just put `game` there.

## That's it!

Thanks for reading! I hope to see you in [part 2]({{< ref "/posts/gamedev-for-beginners-using-odin-and-raylib-2" >}} "Go to next part"), where we will add a player that we can control.

Please leave any questions as comments on the [video version](LINK) of this post. I will reply to some of them in text, but I will also every now and then do a live stream where I reply to questions and take additional questions from the viewers.

Also, if you've enjoyed this series so far and want to support me, then please consider buying my game CAT & ONION on [itch.io](https://zylinski.itch.io/cat-and-onion) or [wishlist it on Steam](https://store.steampowered.com/app/2781210/CAT__ONION/). When you buy on itch.io you also get the full Odin + Raylib source of the game.

Have a fantastic day!

/Karl