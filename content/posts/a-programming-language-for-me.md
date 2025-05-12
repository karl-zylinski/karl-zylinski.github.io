---
title: "A programming language made for me"
date: 2025-05-12T13:04:00+02:00

cover:
  image: "/a-programming-language-for-me/cover.png"
---

In my book [Understanding the Odin Programming Language](https://odinbook.com) I wrote that "Odin incorporates some of my favorite C best practices, straight into the language". But I didn't really elaborate on the details. Let's do that here!

This brings me to talking a bit about a previous job I had. Back in 2021 I worked at a place called [Our Machinery](https://ruby0x1.github.io/machinery_blog_archive/). We were creating a whole game engine in plain C. We used a very comfortable and powerful way to program C.

We relied on concepts such as:

- Custom allocators
- Temporary allocators
- Tracking allocators
- Designated initializers
- Zero is initialized
- Cache friendly programming

While working there, I stumbled upon Odin. I read a bit about it. It seemed to incorporate all these things. In many ways it seemed like a language built around the specific way in which we programmed C at my job. Since I liked that way of programming, it almost seemed like a language built for me!

## Custom allocators

At my job we had implemented our own `Allocator` interface in C. An allocator provides a custom way in which one can do dynamic memory allocations. C programmers are used to `malloc` and `free`. But you can make allocators that provide more advanced allocation strategies. Our `Allocator` interface made it possible to reason about allocators in a uniform way and pass them around to functions.

If a function accepted a parameter of type `Allocator`, then it was a hint that its return value was dynamically allocated.

This is exactly how it works in Odin. But the `Allocator` interface [is built into the language's `base` library collection](https://pkg.odin-lang.org/base/runtime/#Allocator)! This meant that Odin's `base` and `core` libraries support these allocators too. At my job, the `Allocator` interface was only supported in our own code: The C standard library didn't support any of that. But in Odin, my own code _and_ the `core` libraries can reason about custom allocators, making the concept even more powerful.

> In Odin, `core` and `base` are two collections of libraries that come with the compiler. Some refer to them as the "standard library" of Odin. But they are shipped as source with the compiler. You are encouraged to make copies of the packages inside `core`, making it possible to tailor those packages to your own needs. So it's more of a "default library" than a "standard library". There's a sensible default, but you're free do things however you want.

## Temporary allocators

Temporary memory allocators provide a way to do dynamic memory allocations that are only needed for a short while. What's "a short while"? Video games have a very convenient "short while": A single frame.

Gone are the days where you need to manually `malloc` and `free` strings and arrays that you need for a short while. Just use the temp allocator. It'll be gone the next frame!

> Odin lets you choose where to put `free_all(context.temp_allocator)`. That's the line that empties the temp allocator. So in a video game I put it as the last line of the "main game loop".

Again, the `core` library and my code uses the same `Allocator` interface. I can just pass `context.temp_allocator` into any `core` library procedure that accepts an `Allocator` parameter.

## Tracking allocators

Manual memory management may seem hard. How do you know if you're leaking memory?

At my job we had a special tracking allocator that could wrap any other allocator. It recorded when an allocation happened, and recorded when it was deallocated. That way we could display a warning on shutdown, if anything hadn't been deallocated.

This is exactly how the tracking allocator that comes with Odin works. Just plop the code below at the top of your `main` procedure. It'll print a list of memory leaks on shutdown.

```go
track: mem.Tracking_Allocator
mem.tracking_allocator_init(&track, context.allocator)
context.allocator = mem.tracking_allocator(&track)

defer {
	if len(track.allocation_map) > 0 {
		fmt.eprintf("=== %v allocations not freed: ===\n", len(track.allocation_map))
		for _, entry in track.allocation_map {
			fmt.eprintf("- %v bytes @ %v\n", entry.size, entry.location)
		}
	}
	mem.tracking_allocator_destroy(&track)
}
```

> You'll also need to do `import "core:mem"` and `import "core:fmt"` at the top of the file. The code above is from the [Odin overview](https://odin-lang.org/docs/overview/#tracking-allocator).

## Zero is initialized (ZII)

ZII, short for zero is initialized, means that you try to make the zero-value of memory valid in as many situations as possible.

In Odin all variables are automatically zero initialized. Not just integers and floats. But all structs as well. Their memory is filled with zeroes when those variables are created. So if `Some_Type` is a struct, then you can just write the following line to declare and zero-initialize a variable of that type:

```go
x: Some_Type
```

This makes ZII extra powerful! There is little risk of variables accidentally being uninitialized. You can lean on that zero initialization. Also, the whole `core` library of Odin relies on ZII as well. So it feels very natural throughout the whole language and its ecosystem.

> You can skip zero initialization by writing `x: Some_Type = ---`. You rarely need to do so, but it can be a good idea in some specific, performance sensitive situations. It's great that zero initialization is opt out, instead of opt in. That way we get way less bugs due to uninitialized memory.


## Designated initializers

This is a feature built into both C and Odin. The code below will create a variable `x` of type `My_Type`. It'll initialize the field `number` to `7`. Any non-mentioned field will be zero-initialized. This plays very well together with the "zero is initialized" concept.

```go
My_Type :: struct {
	number: int,
	sub_thing: Another_Type,
}

Another_Type :: struct {
	some: int,
	more: f32,
	state: bool,
}

x := My_Type {
	number = 7,	
}
```

## Cache friendly programming

The CPU has some memory inside it that is very fast. It's called a cache. If you keep the cache filled with whatever data the CPU might need next, then your program will run very fast.

At my job we had an entity-component-system (ECS) that used what is known as "Structure of Arrays" (SoA). That's a memory layout that can, in certain circumstances, help fill your CPU cache with relevant data. Anyone who has written SoA data types in C knows it's not very fun.

However, Odin comes with built in SoA support. Just put `#soa` in front of an array declaration. It'll automatically re-arrange the memory layout for you.

As an example, the following code uses the "default layout". Also known as "Arrays of Structures" (AoS):

```go
Person :: struct {
	health: int,
	age: int,
}

people: [128]Person
```

The memory layout of that array looks like this:

```txt
people[0].health
people[0].age
people[1].health
people[1].age
people[2].health
people[2].age
people[3].health
people[3].age
people[4].health
people[4].age
... etc
```

If you add `#soa` in front of `[128]Person`, like so:

```go
Person :: struct {
	health: int,
	age: int,
}

people: #soa[128]Person
```

then the memory layout will instead look like this:

```txt
people[0].health
people[1].health
people[2].health
people[3].health
people[4].health
... and 123 more health items
people[0].age
people[1].age
people[2].age
people[3].age
people[4].age
... and 123 more age items
```

Achieving this in C is manual work. But here you just add `#soa`.

> By the way, I discourage anyone who is making their own video game from making an ECS. It's usually not a good idea. Perhaps it's a good idea for some massive game engines. But for a small project it might just make your code harder to write and thereby your game worse. I feel like people who write an ECS before starting on their gameplay code are people who actually don't want to make games: They went to make general purpose game engines. That's fine, but make sure you're not lying to yourself about what it is you want to do. If you want to make a game, then just make a game. Write the code that you need for the problem at hand, don't pretend to be a giant game engine company.

## Finally: Simplicity

Odin is very simple language. The reason my job used C over C++ was partially because of the simplicity of C. But we did sometimes miss some modern ideas of C++. However, C++ is a massive beast. We didn't want to open that can of worms.

Odin retains the simplicity of C while bringing along some nice modern things, such as generics and (explicit) overloading. But the language is still kept small and simple. And it is meant to remain so. Very few language features have been added to Odin over the last few years, mainly the `core` libraries are getting major changes at this point.

## Not everyone has my programming background

Learning Odin came naturally to me, due to it being so similar to the way in which I wrote C code.

You may come from a different background. If these things are unfamiliar to you, but you still want to learn Odin, then perhaps you'd benefit from reading my book [Understanding the Odin Programming Language](https://odinbook.com). It's meant to be a reader-friendly intro to the language. The book tries to give you the insights I already had when I discovered Odin.

## Thanks for reading!

Why not drop by [my Discord server](https://discord.gg/4FsHgtBmFK)? On there you can ask me questions as well as discuss Odin and game development.

Have a nice day!

/Karl Zylinski
