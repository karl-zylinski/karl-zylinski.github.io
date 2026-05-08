---
title: "Tom's Namespaces"
date: 2026-05-09T00:45:00+01:00
draft: true
---

Tom likes making games. He has worked professionally as a game developer, but he also likes to make games in his free time, as a hobby. He makes these hobby games in a "from scratch" kind of way. This means that he just has a "main" procedure and imports some libraries and builds things as needed.

Recently, Tom has found out about a programming language called "Odin". He finds that it fixed many things he dislikes about C++ and C. He thinks that it looks like a fun language to use for game development.

Soon he is making his first little game. He initially did some experiments using SDL but has now settled on using Raylib. Raylib is a package for doing windowing, rendering, input and audio. All the stuff you need for a game!

After tinkering with his game for a while, he had ended up with a huge file called `world.odin`. In there are lots of procedures! One of them has this signature:

```
world_add :: proc(w: ^World, e: Entity)
```

He felt the need to organize things a bit, perhaps to put all the "world things" into their own little compartment. After all, the Odin language is pretty modern! Tom read about something called "packages" that enables the programmer to split things up in neat ways.

A package is just a subfolder. So Tom moved `world.odin` into a subfolder called `world`. His game then the following folder structure:

game
|
+--world

There was a `main.odin` file in the `game.odin` package from which he then imported the new `world` package, like so:

```
import "world"
```

The stuff from the imported `world` package ended up in a namespace with the same name. Now, in order to call `world_add` he now had to write `world.world_add`. That's silly, so naturally Tom dropped the `world_` prefix, so the procedure just became:

```
add :: proc(w: ^World, e: Entity)
```

Then he could neatly write `world.add(some_world, some_entity)`. But before he got there he went "Oh no! That `Entity` type lives in the main game package!"... So Tom tried to make the `world` package import the `game` package, like so:

```
import game ".."
```

Then he went "Oh no!" again. A compilation error! It says that Odin does not allow cyclic package imports. `game` is importing `world` and `world` is importing `game`.

He then thought that perhaps he needed to create an `entity` package that both `world` and `game` could import, or perhaps a `common` package for "often used types". Oh boy, this is getting a bit hairy!

Tom then hops on the Odin Discord server and starts a discussion on how to actually organize these things. He's told that packages are not meant for fine-grained organization: They are better seen as individual libraries or completely independent parts of a program.

With this in mind, Tom retreats back to his game and tries to move everything back to the `game` package. It did indeed seem like he was over-complicating things a bit. He gently taps his laptop and says to himself, "yeah! keep things simple! I'll just find some other way to namespace things within the `game` package".

Oh no!

Tom looks around a bit and asks again on the Odin Discord: There are no namespacing features in Odin other than the namespace you get when you import packages. He has no way to make a "package-local" namespace that exists only within a package.

Tom gets annoyed and thinks: Other languages have this! Look at C++, there you can write stuff like:

```
namespace world {
	void add( //... etc
}
```
Why can't I have this in Odin? Hah! I'll just make my own fork!

Tom starts sketching on some changes to the Odin compiler to support package-local namespaces.

Then he gets an e-mail: He has gotten a contracting gig! That's great news. The money is running dry. The contract is to help with some C++ game engine programming. It'll take a few weeks, so he puts his Odin experiments on hold for a while.

It was a while since Tom had used C++ much. While working on this big C++ game engine, he already missed a lot of the nice features in Odin.

But at least he'll get to use the nice C++ namespaces! The C++ code base even has an `add` function in a namespace called `world`!

As he was working in another part of the code base, he actually needs to look at the implementation of `add` in the `world` namespace. So he pops up a "find symbol" window and types `add`. He gets 23 hits and it is not very easy for him to find the one he needs.

Oh boy, this codebase really has a lot of namespaces. And it's really big! There are lots of procedure names that appear in lots of namespaces. Tom actually finds it so hard to search for functions in namespaces that he suddenly catches himself with thinking: If I could just type something like `world_add` instead.

He talks to his co-worker Bob about it. Toms says that he finds it a bit annoying to navigate. Bob says that Tom's editor probably isn't very good. A good editor should show the namespace and allow you to fuzzy search using namespace and function name. "It's a tooling problem!" Bob says.

"Too bad most of the tools I've tried don't solve this then..." Tom replies.

When Tom's contract ends, then he returns to his Odin hobby project. He stops thinking about how to add package-local namespaces to Odin. He no longer wants them. In his simple editor he can very quickly find `world_add`.

---

Tom is actually me, Karl Zylinski. I went through this kind of namespace philosophizing period, over a much longer period of time.

The reason I thought that C++ namespaces were good was because I had mostly worked in small C++ projects. When you have a bigger project, then they mostly become annoying. In a big project you must really make sure it is easy to find stuff. And the best way to find stuff is by making sure that things have simple names.

Odin actually has this issue within its `core` libraries: There are lots of procedure names that are common to many packages in `core`. It can sometimes be a bit cumbersome to find the right one. Since I stopped caring for package-local namespaces, I have even sometimes thought that perhaps C is right: Perhaps adding prefixes on the procedure name within libraries is good. But I think there is a compromise to be found here. The jungle of similar names across core is worth it, it having the namespace defined at the import location is nice, as you don't have to worry about name collisions when designing a library.