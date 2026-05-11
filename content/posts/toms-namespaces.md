---
title: "Tom's Namespaces: An Odin Fanfic"
date: 2026-05-11T12:15:00+01:00
cover:
  image: "/toms_namespaces.png"
  caption: "Cover art by [Gerry](https://gerry.website/)"
---

Tom likes making games. He has worked professionally as a game developer, but he also likes to make games in his free time, as a hobby. He makes these hobby games in a "from scratch" kind of way. This means that he just has a `main` procedure and imports some libraries and builds things as needed.

Recently, Tom has found out about a programming language called [_Odin_](https://odin-lang.org/). He found that it fixed many things he disliked about C++ and C. He thought that it looked like a fun language to use for game development.

Soon he is making his first little game. He initially did some experiments using SDL, but has now settled on using Raylib. Raylib is a library for doing windowing, rendering, input and audio. All the stuff you need for a game!

After tinkering with his game for a while, he ended up with a huge file called `world.odin`. There are lots of procedures in there! One of them has this signature:

```
world_add :: proc(w: ^World, e: Entity)
```

He felt the need to organize things a bit, perhaps to put all the "world things" into their own little compartment. After all, the Odin language is pretty modern! Tom read about something called "packages" that enables the programmer to split things up in neat ways.

He read that a package is just a subfolder. So Tom moved `world.odin` into a subfolder called `world`. His game then had the following folder structure:

```txt
game
|
+--world
```

From within the `game` package he could then import the stuff from the `world` package by writing the following Odin code:

```
import "world"
```

The things from this imported `world` package is put into a namespace called `world`. To use anything from in there you'd write `world.some_procedure_in_the_world_package()`. So, in order to call `world_add` Tom had to write `world.world_add`. That's silly, so naturally he dropped the `world_` prefix, so the procedure just became:

```
add :: proc(w: ^World, e: Entity)
```

Then he could neatly write `world.add(some_world, some_entity)`. But before he got there he went "Oh no! That `Entity` type lives in the main game package!"... So Tom tried to make the `world` package import the `game` package, like so:

```
import game ".."
```

Then he went "Oh no!" again. A compilation error! The error says that Odin does not allow cyclic package imports. `game` is importing `world` and `world` is importing `game`.

He then thought that perhaps he needed to create an `entity` package that both `world` and `game` could import, or perhaps a `common` package for "often used types". Oh boy, this is getting a bit hairy!

Tom then hops on the Odin Discord server and starts a discussion on how to actually organize these things. He's told that packages are _not_ meant for fine-grained organization: They are better seen as individual libraries or completely independent parts of a program.

With this in mind, Tom tries to move everything back to the `game` package. It did indeed seem like he was over-complicating things a bit. He gently taps his laptop and says to himself: "Yeah! Keep things simple! I'll just find some other way to namespace things within the `game` package".

Oh no!

Tom looks around a bit and again asks on the Odin Discord. He quickly reaches a conclusion: There are no namespacing features in Odin other than the namespace you get when you import packages. He has no way to make a "package-local" namespace that exists only within a package.

Tom gets annoyed and thinks: Other languages have this! Look at C++, there you can write stuff like:

```
namespace world {
	void add(World *w, const Entity *e) {

	}
}
```
Why can't I have this in Odin? Hah! I'll just make my own fork of the Odin compiler and add it!!

Tom starts sketching on some changes to the Odin compiler to support package-local namespaces.

Before he gets very far, an e-mail arrives: He has gotten a contracting gig! That's great news. The money is running dry. The contract is to help with some C++ game engine programming. It'll take a few weeks, so he puts his Odin experiments on hold for a while.

It was a long time since Tom used C++. While working on this big C++ game engine, he already missed a lot of the nice features in Odin.

But at least he'll get to use the nice C++ namespaces! The C++ code base even has an `add` function in a namespace called `world`!

As he was working in another part of the code base, he actually needed to look at the implementation of `add` in the `world` namespace. So he pops up a "find symbol" window and types `add`. Enter. He gets 23 hits and it is not very easy for him to find the one he needs.

Oh boy, this codebase is really big. At it has a lot of namespaces! There are lots of function names that are reused in many different namespaces. Tom actually finds it so hard to search for functions in namespaces, that he suddenly catches himself with thinking: If I could just type something like `world_add` instead.

He talks to his co-worker Bob about it. Toms says that he finds it a bit annoying to navigate. Bob says that Tom's editor probably isn't very good. A good editor should show the namespace and allow you to fuzzy search using namespace and function name. "It's a tooling problem!" Bob says.

"Too bad most of the tools I've tried don't solve this then..." Tom replies.

When Tom's contract ends, then he returns to his Odin hobby project. He stops thinking about how to add package-local namespaces to Odin. He no longer wants them. In his simple editor he can very quickly find `world_add`.

---

Tom is actually me, Karl Zylinski. I went through this kind of namespace philosophizing period, over a much longer period of time.

The reason I thought that C++ namespaces were good was because I had mostly worked in small C++ projects. When you have a bigger project, then they mostly become annoying. In a big project, you must make sure that it is really easy to find stuff. And the best way to find stuff is by making sure that things have simple names. Here simple doesn't mean "short", but rather that the name is unambiguous.

Tom's co-worker Bob says that these issues are tooling problems. Tom's reply "Too bad most of the tools I've tried don't solve this then..." is important: These things _aren't_ solved by most tools I've tried. At the C++ gig, I saw other people struggle with navigating the codebase too.

There's another thing to consider here: If your codebase requires lots of fancy tools to be explored, then perhaps you don't have a very readable codebase. I would argue that if your code is not possible to navigate in the most simple source code editor, then you have a problem. You can't assume that everyone has the same tools as you.

> Perhaps at a big company you can make everyone use the same tools while working on a product. Although, people tend to start using their personal favorite text editors regardless of what the company thinks.

This problem of generic procedure names being reused a lot actually exists in Odin's `core` collection as well. There are lots of procedure names that are common to many packages in `core`. For example, the procedure name `init` is reused `40` times across all the core packages! It can sometimes be a bit cumbersome to find the right one when using symbol search. Since I stopped caring for package-local namespaces, I have even sometimes thought that perhaps C is right: Perhaps adding prefixes on the procedure names is good, and thus not having the import-site namespacing. That would invalidate the whole package system of Odin. However, I think there is a compromise to be found here. The issue has to be weighed against the fact that Odin's import-site namespacing avoids name collisions, since you can always give an imported package an alias. That way, you don't have to worry about name collisions when designing a library.

It should be noted that a library having its own namespace is a different problem domain compared to using namespaces within a program to organize it. The namespaced library solves a specific problem: When someone uses your library in combination with another library, naming conflicts cannot happen. That justifies the usage of namespaces in that context. Within a program, the code is monolithic, and conflicts are therefore easily fixed. So that use for namespaces isn't relevant. Instead, we end up with making the code harder to navigate. These are the kind of compromises one has to consider. It's funny how long time it can take to gain clarity on these matters.