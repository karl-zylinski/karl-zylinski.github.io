---
title: "Hot Reload Gameplay Code: What, why, limitations and examples!"
date: 2023-09-15T15:47:11+02:00

cover:
  image: "hot_reload_gameplay_code.png"
---

This post is about what hot reload for gameplay code is, why you would use it and what the limitations are. I'll also present an example implementation in [Odin Programming Language](https://odin-lang.org/). The final section is on some caveats, improvements and how to use hot reload in Odin together with [Raylib](https://www.raylib.com/). The front parts are language agnostic while the further down you read the more Odin specific it gets.

The methods I will outline here are for people who are writing their own game 'from scratch', meaning you don't use any game engine.


## What is hot reload and why do you do it?

Hot reloading gameplay code means that you swap out the code that controls the behavior of your game while the game is running. Why? To improve and tweak your gameplay code without having to restart the game.

When making my game [CAT & ONION](https://store.steampowered.com/app/2781210/CAT__ONION/) I had hot reload since the first week. It was of great help. It improved my programming flow a lot by keeping me in the zone while developing new gameplay and editor features. It has made game development more relaxing and fun.

Gameplay programming is one of the most creative types of programming, especially when you're figuring out and tweaking the design while implementing the gameplay, as may often happen for solo developers. Having hot reload helps you in this creative process by minimizing the interruptions to your creative flow.

However, it is important to know when hot reload is helpful and when it falls short.

### Hot reload is helpful when:

- **Working on the behavior of a specific gameplay mechanic.** An example is a melee combat system: How long is the attack you're currently prototyping? What is the range of it? Does it spawn particle effects when it strikes the environment? With hot reload you could change the code for a specific attack, recompile and immediately hit the “attack button” in the game and see how your changes feel.

- **Building UIs for tools or gameplay.** Hot reload works wonderful together with Immediate Mode GUI (IMGUI). This type of UIs are constructed by code running each frame, where that code dictates what kind of UI controls to render, their position, content and size. With hot reload you can both add new UI controls by adding code to draw those, but also do fine tuning of the position, making your UIs look prettier. [Dear IMGUI](https://github.com/ocornut/imgui) is a popular implementation, but you can also [make your own IMGUI](https://www.youtube.com/watch?v=V_phHKr0yRY).

In general, I would say that hot reload is extra helpful when tweaking fine details. Without hot reload you have to restart your game for each small change. When you think you're almost done with a gameplay system you realize you have to tweak a handful of parameters a lot, which could mean 100 restarts of your game. I think that having something like hot reload will make your game better, since you may not have the stamina to do all those restarts.

### Hot reload is *not* helpful when:

- **You make changes to the memory layout of your game.** This means adding fields to structs or reordering fields. This might seem like a big downside, but the biggest frustration hot reload solves is that you don't have to restart when making changes to or fine tuning the behavior of the game. I tend to spend a lot more time on gameplay behavior and algorithms than adding and removing fields from my structs.
- **Your code relies heavily on procedure (function) pointers.** The hot reload will load new game code and kick out the old one. It will keep the same game memory around. Now, if that game memory has pointers to procedures within the old code then you are in trouble. There is more details on this under the heading [Can I patch up procedure pointers that point to the old game DLL?](#can-i-patch-up-procedure-pointers-that-point-to-the-old-game-dll)

## Example implementation in Odin Programming Language

Let's look at an example of how to do this. I will be using [Odin programming language](https://odin-lang.org/), but the general ideas are transferable to languages like C and C++.

**UPDATE:** Since writing this article I've made a an [Odin + Raylib + Hot Reload template](https://github.com/karl-zylinski/odin-raylib-hot-reload-game-template). You can use that one as another example implementation.

In case you prefer watching videos, then I've also made a YouTube video on this topic:

{{< youtube HY6Dv0Fbu14 >}}

Here's the general idea: we split our game into two parts. The first part is a game DLL which contains our game code. The second part is a small program called "the main program" which loads and interfaces with the game DLL and controls the hot reload.

### Part 1: The game DLL

The game DLL contains your whole game. It exposes procedures such as `game_init`, `game_update` and `game_shutdown`. The `game_init` procedure is responsible for allocating what I refer to as the Game Memory. This is a dynamically allocated struct which should be the thing inside which all your game state lives. The central trick the hot reload does is to swap out the game DLL but make sure the new game DLL uses the same memory. Because of this the game DLL also needs to expose a procedure that I call `game_memory`, which returns a pointer to the Game Memory. This way the main program can feed the that pointer back to the newly loaded game DLL.

Here follows the complete code for a bare bones game DLL. The comments within the code are to be seen as part of this article.

```C
package game

import "core:fmt"

/* Our game's state lives within this struct. In
order for hot reload to work the game's memory
must be transferable from one game DLL to
another when a hot reload occurs. We can do that
when all the game's memory live in here. */
GameMemory :: struct {
  some_state: int,
}

g_mem: ^GameMemory

/* Allocates the GameMemory that we use to store
our game's state. We assign it to a global
variable so we can use it from the other
procedures. */
@(export)
game_init :: proc() {
  g_mem = new(GameMemory)
}

/* Simulation and rendering goes here. Return
false when you wish to terminate the program. */
@(export)
game_update :: proc() -> bool {
  g_mem.some_state += 1
  fmt.println(g_mem.some_state)
  return true
}

/* Called by the main program when the main loop
has exited. Clean up your memory here. */
@(export)
game_shutdown :: proc() {
  free(g_mem)
}

/* Returns a pointer to the game memory. When
hot reloading, the main program needs a pointer
to the game memory. It can then load a new game
DLL and tell it to use the same memory by calling
game_hot_reloaded on the new game DLL, supplying
it the game memory pointer. */
@(export)
game_memory :: proc() -> rawptr {
  return g_mem
}

/* Used to set the game memory pointer after a
hot reload occurs. See game_memory comments. */
@(export)
game_hot_reloaded :: proc(mem: ^GameMemory) {
  g_mem = mem
}
```

In order to compile this code as a DLL, save it to file named `game.odin` and then compile it using the command line:
`odin build game.odin -file -build-mode:dll -out:game.dll`

### Part 2: The main program

The main program will load the game DLL and setup the game by calling the `game_init` procedure of the game DLL. It will then go into a main loop and each frame call the game DLL's `game_update` procedure, if that procedure returns `false` then the main loop will terminate. After the update the main loop will check if the time `game.dll` was written (modified) is the same time as when the current game DLL was loaded. If it differs, then it tries to load the newly modified game DLL. If the load succeeds, then the program will fetch the game_memory pointer from the old game DLL using the `game_memory` procedure and pass that to the new game DLL using the `game_hot_reloaded` procedure.

Here follows the full source code for the main program. Note that when loading the game DLL I create a `GameAPI`, which is a container for pointers to the procedures that the game DLL exposes, as well as the 'last written time' the game DLL had etc. The comments are once again to be seen as part of this post.

```C
package game

import "core:dynlib"
import "core:os"
import "core:fmt"
import "core:c/libc"

/* The main program loads a game DLL and checks
once per frame if it changed. If changed, then
it loads it as a new game DLL. It will feed the
new DLL the memory the old one used. */
main :: proc() {
  /* Used to version the game DLL. Incremented
  on each game DLL reload.*/
  game_api_version := 0
  game_api, game_api_ok := load_game_api(game_api_version)

  if !game_api_ok {
    fmt.println("Failed to load Game API")
    return
  }

  game_api_version += 1

  // Tell the game to start itself up!
  game_api.init()

  // same as while(true) in C
  for {
    /* This updates and renders the game. It
    returns false when we want to exit the
    program (break the main loop). */
    if game_api.update() == false {
      break
    }

    /* Get the last write date of the game DLL
    and compare it to the date of the DLL used
    by the current game API. If different, then
    try to do a hot reload. */
    dll_time, dll_time_err := os.last_write_time_by_name("game.dll")

    reload := dll_time_err == os.ERROR_NONE &&
              game_api.dll_time != dll_time

    if reload {
      /* Load a new game API. Might fail due to
      game.dll still being written by compiler.
      In that case it will try again next frame. */
      new_api, new_api_ok := load_game_api(game_api_version)

      if new_api_ok {
        /* Pointer to game memory used by OLD
        game DLL. */
        game_memory := game_api.memory()

        /* Unload the old game DLL. Note that
        the game memory survives, it will only
        be deallocated when explicitly freed. */
        unload_game_api(game_api)

        /* Replace game API with new one. Now
        any call such as game_api.update() will
        use the new code. */
        game_api = new_api

        /* Tell the new game API to use the old
        one's game memory. */
        game_api.hot_reloaded(game_memory)

        game_api_version += 1
      }
    }
  }

  // Tell game to deallocate its memory.
  game_api.shutdown()
  unload_game_api(game_api)
}

/* Contains pointers to the procedures exposed
by the game DLL. */
GameAPI :: struct {
  init: proc(),
  update: proc() -> bool,
  shutdown: proc(),
  memory: proc() -> rawptr,
  hot_reloaded: proc(rawptr),

  // The loaded DLL handle
  lib: dynlib.Library,

  /* Used to compare write date on disk vs when
  game API was created. */
  dll_time: os.File_Time,
  api_version: int,
}

/* Load the game DLL and return a new GameAPI
that contains pointers to the required
procedures of the game DLL. */
load_game_api :: proc(api_version: int) -> (GameAPI, bool) {
  dll_time, dll_time_err := os.last_write_time_by_name("game.dll")

  if dll_time_err != os.ERROR_NONE {
    fmt.println("Could not fetch last write date of game.dll")
    return {}, false
  }

  /* Can't load the game DLL directly. This
  would lock it and prevent hot reload since the
  compiler can no longer write to it. Instead,
  make a unique name based on api_version and
  copy the DLL to that location. */
  dll_name := fmt.tprintf("game_{0}.dll", api_version)

  /* Copy the DLL. Sometimes fails since our
  program tries to copy it before the compiler
  has finished writing it. In that case,
  try again next frame!

  Note: Here I use Windows copy command, there
  are better ways to copy a file. */
  copy_cmd := fmt.ctprintf("copy game.dll {0}", dll_name)
  if libc.system(copy_cmd) != 0 {
    fmt.println("Failed to copy game.dll to {0}", dll_name)
    return {}, false
  }

  // Load the newly copied game DLL
  lib, lib_ok := dynlib.load_library(dll_name)

  if !lib_ok {
    fmt.println("Failed loading game DLL")
    return {}, false
  }

  /* Fetch all procedures marked with @(export)
  inside the game DLL. Note that we manually
  cast them to the correct signatures. */
  api := GameAPI {
    init = cast(proc())(dynlib.symbol_address(lib, "game_init") or_else nil),
    update = cast(proc() -> bool)(dynlib.symbol_address(lib, "game_update") or_else nil),
    shutdown = cast(proc())(dynlib.symbol_address(lib, "game_shutdown") or_else nil),
    memory = cast(proc() -> rawptr)(dynlib.symbol_address(lib, "game_memory") or_else nil),
    hot_reloaded = cast(proc(rawptr))(dynlib.symbol_address(lib, "game_hot_reloaded") or_else nil),

    lib = lib,
    dll_time = dll_time,
    api_version = api_version,
  }

  if api.init == nil || api.update == nil || api.shutdown == nil || api.memory == nil || api.hot_reloaded == nil {
    dynlib.unload_library(api.lib)
    fmt.println("Game DLL missing required procedure")
    return {}, false
  }

  return api, true
}

unload_game_api :: proc(api: GameAPI) {
  if api.lib != nil {
    dynlib.unload_library(api.lib)
  }

  /* Delete the copied game DLL.

  Note: I use the windows del command, there are
  better ways to do this. */
  del_cmd := fmt.ctprintf("del game_{0}.dll", api.api_version)
  if libc.system(del_cmd) != 0 {
    fmt.println("Failed to remove game_{0}.dll copy", api.api_version)
  }
}
```

Save this code as `main.odin` and compile it using:
`odin build main.odin -file`

**UPDATE:** Instead of the manual `init = cast(proc())(dynlib.symbol_address(lib, "game_init") or_else nil)` API proc fetching, you can use `dynlib.initialize_symbols` to match struct members to symbols in the DLL. [See these lines in my hot reload template on GitHub for more info](https://github.com/karl-zylinski/odin-raylib-hot-reload-game-template/blob/24bd982f2e441a8c0e5be7409a393f56b2641885/main_hot_reload/main_hot_reload.odin#L66).

### Do the hot reload!

Start the program by running `main.exe`. You'll see a console start up, it will be printing a constantly increasing number. As a first test of the hot reload, keep the main program running and open `game.odin` in your code editor. Within the `game_update` procedure change the `+=` to `-=`, so that the number will decrement each frame. Save the file and again compile the game DLL again using:
`odin build game.odin -file -build-mode:dll -out:game.dll`
(make sure you don't close the running `main.exe`)

The main program will detect that the game DLL changed and reload it. You should almost instantly after the compile see how the counter starts decreasing each frame.

That's the basics! From here on I will go into some caveats and good things to know.

## Some common questions & good improvements

### Can I do full reset of the game?

While you could fully reset your game by turning the main program off and on again, it is quite useful to have an in-game way to completely reset it without having to restart the program. This is useful if you have ended up in a bad state or just want to start over your play testing. I do something like follows.

In game.odin, add a procedure called `game_reset` that returns a `bool`. Return `true` when you want a full reset to happen, you could for example check if a certain hotkey is pressed and in that case return `true`. Make it a part of the `GameAPI` struct and make sure to fetch it during `load_game_api`. Then, change your main loop to:

```C
for {
  if game_api.update() == false {
    break
  }

  dll_time, dll_time_err := os.last_write_time_by_name("game.dll")

  full_reset := game_api.full_reset()
  reload := full_reset ||
    (dll_time_err == os.ERROR_NONE &&
     game_api.dll_time != dll_time)

  if reload {
    new_api, new_api_ok := load_game_api(game_api_version)
    if new_api_ok {
      if full_reset {
        game_api.shutdown()
        unload_game_api(game_api)
        game_api = new_api
        game_api.init(level_filename)
      } else {
        game_memory := game_api.memory()
        unload_game_api(game_api)
        game_api = new_api
        game_api.hot_reloaded(game_memory)
      }

      game_api_version += 1
    }
  }
}
```

The difference compared to earlier is that we ask the Game API if we should do a full reset. If we should, then we run `game_api.shutdown()` clearing the memory, after which we unload the old game API. We then run `game_api.init()` again. This is like restarting the game, but without having to restart the exe. It doesn't have anything to do with hot reload really, but it is handy.

### Can I avoid crashes when struct layouts change?

If you add or remove from `GameMemory` a reload occurs, then the code in the game DLL might end up using the wrong memory for the wrong thing. This can lead to weird behavior or in worst case a crash. In the case of weird behavior you'll probably notice it, so then you can just issue a full reset (as described in the previous section).

However, you cannot avoid all crashes. One trick you can do is to compare the size of the `GameMemory` struct before and after the reload. If those sizes differ then you could issue a full reset, just as a precaution. In order to know the size of the `GameMemory` you could add a procedure to the game DLL that simply returns `size_of(GameMemory)`. Note that if `GameMemory` contains a pointer to a some dynamically allocated struct, then this will not help for when that struct's layout changes.

Another idea that could help is to stop compiling the game DLL externally. Instead you could have two hotkeys in your game: One that issues a recompile by invoking the Odin compiler and then follow it up by hot reloading the DLL. The other hotkey could also do the compile but do a full reset instead of a hot reload. This way you could press the full reset button when you know you've changed struct layouts (also it feels pretty funky to make your game compile and reload itself).

Given that you mostly use the hot reload to change procedures and tweak your game's behavior, the crashes aren't much of a problem. If it does crash, then you just have to restart it like in the old days! It doesn't happen often and the time saved from having hot reload is great regardless.

Some people try to implement serialization and deserialization of all the state when the hot reload happens in order to 'transfer' the state to the new game DLL. I do not think this is worth it. It is important to know where the bottlenecks in your workflow are. Doing something that complicated to all your state probably has negligible returns. It's important to remember that you are building a game and that you should only implement systems and tools that really help you, not because it's technologically interesting.

### My global variable dies when I hot reload!

If you put a global variable inside `game.odin` and hot reload then that global variable will be reset its initial state when the new game DLL is loaded. This can be fixed by
only using global variables that are pointers to fields within `GameMemory`. When a hot reload occurs and `game_hot_reloaded` is run, you can then reassign those global pointers to the correct memory inside the `GameMemory` struct.

In code this would look something like:

```C
SomeStruct :: struct {
  // important state
}

GameMemory :: struct {
  some_struct: SomeStruct,
}

g_some_struct: ^SomeStruct
g_mem: ^GameMemory

@(export)
game_hot_reloaded :: proc(mem: ^GameMemory) {
  g_mem = mem
  g_some_struct = &g_mem.some_struct
}
```

I do this in my game. There is a global pointer to the current world. But that world actually lives inside `GameMemory` and is reassigned when `game_hot_reloaded` is run.

### Can I patch up procedure pointers that point to the old game DLL?

When the old game DLL is unloaded and you've stored pointers to procedures within `GameMemory`, then you are in trouble. The DLL those procedures live in will get unloaded and your procedure pointers will be broken. Here are a couple of ideas on how to tackle this:

1. Don't unload old game DLL in your main program. This will make those procedures still stay around. But if you make changes to the procedures being pointed to, then it will still use the old code, which is not ideal. This doesn't really solve the problem of hot reload not working for procedure pointers, but it makes sure your program doesn't crash.
2. In the `game_hot_reloaded` procedure, try to find all the places where you have procedure pointers and patch them up again. This is harder than it sounds since you might have dynamic arrays of structs that contain procedure pointers where it is unclear where they came from.
3. Only use procedure pointers when you set up well-defined objects such as structs that define APIs. In this case you re-setup those APIs in `game_hot_reloaded`. For example, an abstract renderer that has both Direct3D and Vulkan backends might use procedure pointers inside some `RendererAPI` struct. You could keep track of all such APIs and reload them during `game_hot_reload`, but otherwise avoid procedure pointers.
4. Simplify your code by trying to use enums instead of procedure pointers. This will make your code easier to follow and also fix this issue. You can then switch on those enums and call the correct, up-to-date procedure. You can simplify your game a lot by just using enums to guide your gameplay branching. Many of the places where you thought you needed procedure pointers will work just as well with enums, and the code will be easier to read too!

### Release build without hot reload

When you ship your game you probably don't want to include the hot reload stuff. For a release build you could create a separate main_release program that skips all the hot reload stuff and directly includes the game package. Something like this:

```C
package main_release

import "../game"

main :: proc() {
  game.game_init()

  for {
    if game.game_update() == false {
      break
    }
  }

  game.game_shutdown()
}
```

In our earlier examples we compiled using `odin build main.odin -file`, i.e. we hadn't separated things into packages, instead we compiled specific source files directly. Now we need the code that goes into the game DLL to live in a separate package it can be imported into our main_release program. Therefore we would in this example split everything into three packages that live in folders named game, main and main_release. You'd compile the game DLL by running `odin build game -build-mode:dll -out:game.dll` and the main program by running `odin build main` and finally the main_release program by running `odin build main_release`.

Note: The main_release program directly imports the game code. The game.dll is not needed for this type of release build, it's all compiled into a single exe.

### I can't recompile game DLL with debugger attached

When Visual Studio is attached to main.exe and you try to recompile you'll notice that it fails due to the game.pdb being locked. My workaround is to use the [RAD Debugger](https://github.com/EpicGames/raddebugger) instead of Visual Studio or VSCode. It works perfectly well with Odin and with it you'll be able to hot reload and keep the debugger attached. It is alpha software however, so it has some weird things about it.

### Using hot reload in Odin together with Raylib

I like using Odin combined with Raylib. Raylib lets me draw graphics and play sound, sort of like SDL. Hot reload works well with Raylib, but there are a few things you need to do.

#### Using the DLL version of Raylib

First off, Raylib stores its state internally as global variables. By default, the Raylib package that is shipped with Odin is linked statically. This means that the game DLL will own the Raylib package's global state. So when you hot reload the game DLL, all the Raylib state is cleared and your program probably crashes. This is fixed by using the DLL version of Raylib instead. In order to do this, I recommend the following:

1. Go to where your Odin compiler lives. Go into the `vendor` folder. Copy the whole `raylib` folder to your game's folder.
2. Open your game's local copy of `raylib/raylib.odin`
3. Change the line
    `@(extra_linker_flags="/NODEFAULTLIB:libcmt")` to `@(extra_linker_flags="/NODEFAULTLIB:msvcrt")` and the line `"windows/raylib.lib",` to `"windows/raylibdll.lib",` (this of course is only for Windows, I'm sure you can do something similar on Linux and Mac)
4. Instead of doing `import rl "vendor:raylib"` just do `import rl "raylib"` to use your game's own copy of Raylib.

You'll also need to copy the `raylib.dll` file inside `raylib/windows` to where your exe lives.

#### Split `game_init` into `game_init` and `game_init_window`

In your game DLL you have an `game_init` procedure. Now, you could create your Raylib window in there. But then, whenever you do a full reset, it would try to create another window! A solution is to have a `game_init_window` and `game_shutdown_window` procedure exposed by the game DLL in which you open and close your Raylib window respectively. You'll only run the `game_init_window` procedure before the main loop and the `shutdown_window` procedure after the main loop finishes.

#### Is the Raylib DLL lifetime bound to game.dll?

The Raylib DLL is automatically loaded when game.dll is loaded by the main program. So you may wonder when the Raylib DLL is unloaded. The answer is that the Raylib DLL is implicitly pulled in by game.dll, but unloading game.dll does not unload the implicitly pulled in Raylib DLL. So you do not have to worry that the Raylib DLL and its internal global state gets trashed when game.dll is reloaded.

## That's it!

Thanks for reading! Hot reload has improved the flow of my creative coding a lot. I hope it will help you too!

If you liked this article and want to support me, then please consider becoming a patron: https://www.patreon.com/karl_zylinski

Drop any comments or questions to me on:

- Twitter: https://twitter.com/karl_zylinski
- Mastodon: https://mastodon.gamedev.place/@karl_zylinski
- E-mail: karl@zylinski.se

