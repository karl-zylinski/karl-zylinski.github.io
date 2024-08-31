---
title: "Solodevs and the trap of the game engine"
date: 2024-05-23

cover:
  image: "/winding_path.png"
---

Since I was 19 I've wanted to make my own video games, as in creating everything in the game myself, often referred to as being a solo developer, or solodev. But I was bad at committing to anything and I always thought that I lacked some specific skill such as music making, game design skills and art skills.

Now I'm 35 and just released my own game (see trailer below). I've made everything in it myself.

<figure>
{{< youtube gggtSXiJkKc >}}
<figcaption>Trailer for my game CAT & ONION, the first game I've made all by myself</figcaption>
</figure>

What happened in the time inbetween 19 and 35, and what stopped me from making my own game? Well, lots of things happened. I went to university, I got a job in the games industry (engine tools programmer), I worked at several game companies. I got burnt out. Studied astrophysics. Then back to games again. And then finally I ended up trying to make a game by myself and actually managed to finish it.

The funny thing is: My approach to making games at 35 is more similar to the one I had around age 19 than the approach I learned from working in the games industry.

## The approach when I was around 19

When I was somewhere between 19 and 22 I used to make games using C++ and the library known as [SFML](https://www.sfml-dev.org/). This just meant creating a C++ program that used SFML to open a window, check for some input and draw some graphics. There was no big game engine, just a C++ program that used SFML in a very ad-hoc way to get things done.

At the time, I never actually finished any game using this method, other than some group projects in school. But some of my prototypes got pretty far!

## Getting a job in the games industry

Around age 23 I got a job as a game engine tools programmer. I worked with some very talented people on the Bitsquid game engine (it's discontinued, but still used for creating game such as Helldivers 2). My job was to create editors for the engine, such as level editors etc. I also got a lot of insight into the C++ code base of the engine.

Naturally I adopted lots of the mindsets of how to make games from this job. I.e. I started thinking that before you make a game you need to have a nice, shiny engine in place with good tools.

## Where things went a bit wrong

It is very easy to look at a general purpose game engine such as Unity or Godot and think that you should use such engines with such tools to create games. Now, using such engines is perfectly fine. But the problem is when you want to create your own games from scratch, then you might get the idea that you also need to build a general purpose game engine before you get started with the gameplay code.

I still wanted to create my own games, as in solo developing them. So in my spare time I was tinkering away. But I got way too inspired by my day job, where I was developing a general purpose game engine! I was creating renderers, abstract level editing tools, file formats, data compilers. But I didn't write any gameplay code _at all_! That, I thought, came when the engine code was mature enough.

## Using third-party game engines

In my twenties I did lots of game jams, and I made many of them Unity. It was usually a quite enjoyable process. But whenever I tried to take those games beyond the gamejam / prototype stage, it just stopped being fun.

Engines like Unity are data-heavy, in the sense that you create some stuff in the editor and then hook those things up with some code. It's a jumble where you jump back and forth between editors and code.

Having those kinds of tools are good for empowering non-technical game designers to create games. But for a designer/programmer who tries to make a game by themselves, this split mostly introduces friction and (at least to me) makes the creation of games less fun.

## Odin + Raylib: Back to basics

A few years ago I ended up trying the [Odin Programming Language](https://odin-lang.org/) along with [Raylib](https://www.raylib.com/). Raylib is a library similar to SFML, but written in C. Odin comes with built in bindings for Raylib.

When I played around with this combo I had a flashback: This feels like the good fun days when I made games using C++ and SFML. Not because of any similarity between the languages and libraries, but because my game was just a program that used a lib to show some graphics.

I could quickly get some stuff to display on the screen, but in contrast to using a third party game engine I could put as much stuff in code as I wanted, there was no dependency on data created within game engine tools.

I have had to create some tools for editing levels and some other data that is too hard to describe in code. But I only had to do so out of necessity. Sometimes I even scrapped a tool I made when I realized I could describe the same stuff using code. My idea these days is this: I am a solo developer, I don't need to put stuff into friendly data formats or tools unless the tool really aids my creation process. Put as much in the code as possible. A procedure with some parameters for creating certain types of game objects is often better than an editor.

If you work on a team you must make sure to provide tools to empower the team, but for a solo developer, creating tools that you actually don't need, might just have the opposite effect.

Again, comparing my 35 year old self to my 19 year old self, I found an advantage to using Odin over C++. My C++ projects usually didn't leave the prototype stage. Perhaps it was due to inexperience. But also, there is a certain feeling of dread sometimes when you have a big C++ code base and want to implement a new mechanic, because C++ is a bit too chatty. There is just slightly too much friction in C++. In Odin the code is both more compact and simpler, making it possible for me to (just barely) clear the mental hill of having to add new features when I need to.

Also Raylib is a bit simpler to use than SFML. All-in-all it just felt _fun_ to write games again, and I felt like I could get to writing gameplay very quickly and also have an easier overview of the project than if I use a big engine like Unity.

## Was working with creating game engines a mistake?

No. I learned very good programming principles while professionally making game engines and game engine tools. The mistake was that I started to think that in order to make a game from scratch, you must first create a general purpose game engine.

For me, the rule is: Make games, not general purpose game engines.

## Thanks for reading!

If you want to see some game development streams and tutorials, then maybe you'll enjoy my YouTube channel: https://www.youtube.com/@karl_zylinski

/Karl Zylinski