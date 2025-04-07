---
title: "Dynamic Arrays in Arenas"
date: 2025-04-04T12:29:00+02:00

cover:
  image: "arena-mistakes/cover.png"
---

When programming in Odin you can use arena allocators. If you use an arena allocator combined with a dynamic array, then there are a couple of pitfalls that may not be apparent at first. Let's look at what arenas are, how you can run into trouble when naively using them with dynamic arrays and what you can do instead.

## What's an arena? How does it work?

Arenas and arena allocators are useful for _grouping_ allocations that have the same lifetime. Here _lifetime_ means allocations that are fine to deallocate at the same time, which can be done by destroying the arena they were allocated into.

> The _arena_ is the thing that holds the memory and the _arena allocator_ says how to allocate into the arena. The allocator is of type `runtime.Allocator`, so that you can pass it around to things that expect an allocator.

There are several different implementations of arenas in Odin. Here's a list of some of them:

- In `core:mem`: `mem.Arena`, `mem.Dynamic_Arena`
- In `core:mem/virtual`: `virtual.Arena` (has three modes of operation: growing, static, buffer)
- In `base:runtime`: `runtime.Arena` (only used by temp allocator, which is a kind of arena allocator)

To keep it simple, let's talk about `mem.Arena`. That one you just feed a pre-allocated block of memory. The arena allocator will allocate into that block. If the pre-allocated block become full, then nothing more can be allocated into it. Here's a simple example:

```go
package arena_example

import "core:fmt"
import "core:mem"

main :: proc() {
	arena_mem := make([]byte, 1*mem.Megabyte)
	arena: mem.Arena
	mem.arena_init(&arena, arena_mem)
	arena_alloc := mem.arena_allocator(&arena)

	number1 := new(int, arena_alloc)
	number2 := new(int, arena_alloc)

	fmt.printfln("Arena memory starts at address %p (%i)", &arena_mem[0], &arena_mem[0])
	fmt.printfln("number1 allocated at address %p (%i)", number1, number1)
	fmt.printfln("number2 allocated at address %p (%i)", number2, number2)

	// Destroys the arena
	delete(arena_mem)
}
```

We make a slice of 1 megabyte of memory and initialize the arena with it. Thereafter we dynamically allocate two variables of type `int` using the arena allocator. Those two `int`s will be allocated into that 1 megabyte block of memory.

If you run the program, it'll print something like:
```txt
Arena memory starts at address 0x206B51F7048 (2227831795784)
number1 allocated at address 0x206B51F7048 (2227831795784)
number2 allocated at address 0x206B51F7050 (2227831795792)
```

As you can see `number1` has the same address as the first byte of the arena's memory block: It sits right at the beginning of the arena's memory block. The address of `number2` is `8` bytes after `number1`.

![Shows how number1 and number2 end up one-after-the other in the arena's memory, each takes 8 bytes](/arena-mistakes/arena_memory.png)

What we can understand from this is that this arena allocates in a _linear_ fashion. The things we allocate go into the arena's memory block one-after-another, in the order they are allocated. The size of an `int` is `8` bytes (on 64 bit machines), so `number2` ends up `8` bytes from the start of the arena, right after `number1`.

Note that it is _not_ possible to do `free(number1)` or `free(number2)`. That will just return the error `Mode_Not_Implemented`. You can only deallocate everything in the arena, not individual parts of it. This makes sense, since the arena is meant for _things that have the same lifetime_. In the case of `mem.Arena`, you deallocate arena by deleting the memory block itself:

```
delete(arena_mem)
```

For the arenas in the `core:mem/virtual` package you'd use a specific procedure to destroy the arena, such as `virtual.arena_destroy(&some_arena)`. I'll get back to an example involving the virtual memory arenas later.

## Dynamic array trouble

Consider the following small program. It uses an arena allocator combined with a dynamic array.

```go
package dynamic_array_mistake

import "core:fmt"
import "core:mem"
import "core:math/rand"

main :: proc() {
	arena_mem := make([]byte, 1*mem.Megabyte)
	arena: mem.Arena
	mem.arena_init(&arena, arena_mem)
	fmt.printfln("Arena starts at address: %p (%i)", &arena_mem[0], &arena_mem[0])

	arena_alloc := mem.arena_allocator(&arena)
	dyn_arr := make([dynamic]int, arena_alloc)
	append(&dyn_arr, 7)
	fmt.printfln("After 1 append, address of first element is: %p (%i)", &dyn_arr[0], &dyn_arr[0])

	for i in 0..<9999 {
		append(&dyn_arr, rand.int_max(100000))
	}

	fmt.printfln("After 10000 appends, address of first element is: %p (%i)", &dyn_arr[0], &dyn_arr[0])

	// Destroys the arena
	delete(arena_mem)
}
```

We again initialize a `mem.Arena` with a 1 megabyte block of memory. After that we print the address of where the arena begins. On my computer it prints:
```txt
Arena starts at address: 0x253D94C9048 (2559151214664)
```

We then create a dynamic array that can allocate into the arena. We append a single item into the dynamic array. If we print the address of the first element, then it says:

```txt
After 1 append, address of first element is: 0x253D94C9048 (2559151214664)
```

That's the same as our arena. So the dynamic array starts right at the beginning of the arena.

Now we add `9999` more items. That will make the dynamic array grow several times. Thereafter we print the address of the first element again:
```txt
After 10000 appends, address of first element is: 0x253D94E8D48 (2559151344968)
```

That's not the beginning of the arena! How far away from the beginning is it? Subtract the two numbers to find out: `0x253D94E8D48 - 0x253D94C9048 = 0x1FD00`, or in decimal: `130304`. So the first element of the dynamic array is `130304` bytes from the start of the dynamic array.

## What happened?

![Shows how the memory usage of a dynamic array after 1 append and after 9 appends. After 1 append it sits at the start of the arena. After 9 appends it sits further away from the start and it has left the old data as a hole in the dynamic array](/arena-mistakes/growing_dyn_arr.png)

Initially the dynamic array has no memory allocated at all. After 1 append it will have allocated some initial memory. This initial memory will have capacity for 8 items. That's the default for dynamic arrays. Imagine that 8 more items are then appended. This will make the dynamic array run out of capacity, forcing it to grow again. The new capacity will probably be something like 24.

Each time the dynamic array grows, it does this:
- Tell arena allocator to allocate memory for new data block: OK! It gives you that many bytes (as long as it is not full).
- Copy old data to new data block: OK!
- Tell allocator to deallocate the old data: Nope! Arena allocators don't implement individual allocations. This step will have no effect.

Internally, when the dynamic array tries to free the old block, the allocator reports an `Mode_Not_Implemented` error. The dynamic array does not care about that error.

So the old memory block is just left in the arena. Think of what happens if the dynamic array grows several times: You'll waste _a lot_ of memory. It's just a graveyard of old blocks in a trail behind the currently used block. That's why we have `130304` bytes between the start of the arena and the first item of the dynamic array.

## Why can't the arena just deallocate the old block?

As I showed earlier, this kind of arena grows linearly. It just starts at the beginning of the arena's memory block. When allocations happen, it gives out a pointer to the current position in the arena and moves forward by how many bytes you wanted to allocate. The next allocation will happen at the new position.

It's a simple allocator: It just knows where to take memory from and how much memory it has left. It does not keep track of all the previous allocations.

So individual deallocations can't happen: It does not keep track of enough information to do that.

And even if it did track that information, doing deallocations in a linear block of memory like that would quickly lead to _memory fragmentation_: Imagine if you allocate using `new(Some_Type, arena_allocator)` and mix allocations of different sizes. Any deallocation would leave a hole of that size. Quite soon you'd have many tiny unusable "holes" in the memory.

## It's all about lifetimes

In our example the dynamic array has its old block of memory in the arena as well as a new block. It tries to deallocate the old block but keep the new block. The fact that it even tries to do this shows that these two things have different lifetimes!

This is another reason why individual allocations don't make any sense when working with arenas. Since everything in an arena should share the same lifetime, there is only one deallocation that makes sense: Deallocating everything in the arena.

This is the whole point of the arena: You put things into it that should all be destroyed at the same time. It's not a silver bullet for making manual memory management "magically easy". It's just a way to group allocations so you don't have to do lots of separate `free` and `delete` calls in order to clean up.

## Alternatives: Use default allocator

In general, just use the default allocator `context.allocator` with your dynamic arrays. Then they can grow and deallocate their old memory as expected. If you have trouble with their memory leaking, then use the tracking allocator in your development build. Set it up like this: https://odin-lang.org/docs/overview/#tracking-allocator

If you have trouble with the memory of the dynamic array moving, then you can look at the section about the Virtual Growing Arena, below.

## Alternatives: Preallocate with maximum size

If you know the maximum number of things that can go into the dynamic array, then you can do something like this:

```go
dyn_arr := make([dynamic]int, 0, 2000, arena_alloc)
dyn_arr.allocator = mem.panic_allocator()
```

This will construct a dynamic array with capacity for `2000` elements. That amount of memory will immediately get allocated into the arena. The `0` is the _length_ of the dynamic array. That's how many of those `2000` elements that are used.

This means that you can `append` as usual into this dynamic array. Note the line `dyn_arr.allocator = mem.panic_allocator()` -- This will make your program panic (crash on purpose) if the dynamic array tries to grow, as that would litter the arena in the ways we've talked about. If you run into that crash, then perhaps you should increase the capacity.

If the size of the dynamic arrays in your program vary wildly depending on what the user does, then perhaps you should not use an arena. An example is if you're making a video editing software: Some users may use 10 megabytes of memory, while others may use 200 gigabytes, depending on their project sizes. Pre-allocating for the worst-case scenario in these cases is probably not a great idea. The software is used in a very dynamic way, so you'll have to be more dynamic with your memory usage.

## The virtual growing arena

A very useful kind of arena is the growing virtual arena. It's arena that uses blocks of memory, when the current block is full, it allocates a new one and does allocations into that.

> Virtual memory can be _reserved_ without causing the actual memory usage of the computer to increase. A virtual arena allocator can _commit_ parts of that reserved memory, in order to map it to actual memory.
> My book [Understanding the Odin Programming Language](https://odinbook.com) goes into virtual memory arenas in more depth.

If you re-write our initial sample using a virtual growing arena, it would look like this:

```go
package dynamic_array_virtual

import "core:fmt"
import vmem "core:mem/virtual"
import "core:math/rand"

main :: proc() {
	arena: vmem.Arena
	arena_alloc := vmem.arena_allocator(&arena)
	dyn_arr := make([dynamic]int, arena_alloc)
	append(&dyn_arr, 7)
	fmt.println("After 1 append to dynamic array, address of first element is:", &dyn_arr[0])

	for i in 0..<9999 {
		append(&dyn_arr, rand.int_max(100000))
	}

	fmt.println("After 10000 appends to dynamic array, address of first element is:", &dyn_arr[0])
}
```

However, if you run this program, it would print this:
```txt
After 1 append to dynamic array, address of first element is: 0x1A682440038
After 10000 appends to dynamic array, address of first element is: 0x1A682440038
```

Wait a minute! It still has the same address after the 10000 appends! Is the growing virtual arena magical? Does it support individual deallocations somehow?! The first element seems to always sit at the same address.

There's nothing magical going on. This behavior is due to a special case in the virtual growing arena. It reuses the same address if that allocation is still the most recent one into the arena.

> The `Arena` in `core:mem` does not have this special reuse-address-behavior. Perhaps it'll get added in the future, in which case the earlier examples that used `mem.Arena` would start behaving like this one.

So this would not work if you had done any allocation into the arena between the moments when the dynamic array grew. It would have to move further on into the arena, and in so doing leave behind the old data.

Is there then a good reason for doing this? Yes, because the elements in the dynamic array will not move in memory when it grows.

Is there a catch? Yes. If the dynamic array grows past the block size of the arena, then it will have to move to a new block. In the example below I've added the line `arena_err := vmem.arena_init_growing(&arena, 4000)`. This will initialize the growing arena with a initial block size of `4000 bytes`. The contents of the dynamic array will eventually go past that size. That will cause a new block to be created, and the dynamic array will be moved to it. In code:

```go
package dynamic_array_virtual

import "core:fmt"
import vmem "core:mem/virtual"
import "core:math/rand"

main :: proc() {
	arena: vmem.Arena
	arena_err := vmem.arena_init_growing(&arena, 4000)
	assert(arena_err == nil)
	arena_alloc := vmem.arena_allocator(&arena)
	dyn_arr := make([dynamic]int, arena_alloc)
	append(&dyn_arr, 7)
	fmt.println("After 1 append to dynamic array, address of first element is:", &dyn_arr[0])

	for i in 0..<9999 {
		append(&dyn_arr, rand.int_max(100000))
	}

	fmt.println("After 10000 appends to dynamic array, address of first element is:", &dyn_arr[0])
}
```

The program above would print:
```txt
After 1 append to dynamic array, address of first element is: 0x2745AB00038
After 10000 appends to dynamic array, address of first element is: 0x2745AD30038
```
As you can see the address changed. So due to the small block size we are back to the dynamic array moving around within the arena.

> Note: WASM does not support virtual memory. If you need a growing arena on WASM, then you can use `Dynamic_Arena` from `core:mem`. That one also uses blocks, but they are not virtually allocated, so the blocks always use as much physical memory as they are big, even when they are not full. I use `Dynamic_Arena` as a fallback for WASM in my Handle Map implementation: https://github.com/karl-zylinski/odin-handle-map

## The static virtual arena

Similarly to our previous example of using the `panic_allocator`, you could also use a _static virtual arena_ to make it an error if the dynamic array tries to grow past the current block size. The static virtual arena also uses virtual memory, but it only has a single block. In code:

```go
package dynamic_array_virtual_static

import "core:fmt"
import vmem "core:mem/virtual"
import "core:math/rand"

main :: proc() {
	arena: vmem.Arena
	arena_err := vmem.arena_init_static(&arena, 4000)
	assert(arena_err == nil)
	arena_alloc := vmem.arena_allocator(&arena)
	dyn_arr := make([dynamic]int, arena_alloc)
	append(&dyn_arr, 7)
	fmt.println("After 1 append to dynamic array, address of first element is:", &dyn_arr[0])

	for i in 0..<9999 {
		_, err := append(&dyn_arr, rand.int_max(100000))
		fmt.assertf(err == nil, "Error when adding to dynamic array: %v", err)
	}

	fmt.println("After 10000 appends to dynamic array, address of first element is:", &dyn_arr[0])
}
```

The code above would error with:
```txt
Error when adding to dynamic array: Out_Of_Memory
```
Because of the `append` trying to grow past the end of the arena. In other words, it is able to reuse the same address over and over, but eventually fills the block. Then it must move. But it can't, since a static virtual arena only has one block.

However, this number `4000` that I feed into `arena_init_static` is _tiny_ when we talk about virtual memory. Each application has the whole 64 bit virtual addressing space to use. So you could easily use something like `1*mem.Gigabyte` here and never worry about a reallocation. The reserved amount will not make your program's memory usage go up: Only when the memory is actually _committed_, meaning that it is used for an actual allocation, does the usage go up. This committing is done in chunks called _pages_. A page is `4096` bytes on many systems. This is also why the default block size of the _growing_ virtual arena is fairly large: It's 1 megabyte. It reserves blocks of that size and then fills them by committing it as the arena allocator asks for more and more memory.

## You can also skip dynamic memory completely

If you don't use dynamic memory, then you never have deallocate anything.

See `Small_Array` for a dynamic-array-like structure: https://pkg.odin-lang.org/core/container/small_array/

See Jakub's "Static Data Structures" for a bunch of different data structures that don't use dynamic memory at all: https://github.com/jakubtomsu/sds

## Thanks for reading!

If you enjoyed this article, then perhaps you'd enjoy my book ["Understanding the Odin Programming Language"](https://odinbook.com). Read a free sample [here](https://odinbook.com/sample.html).

Ask questions on [my Discord server](https://discord.gg/4FsHgtBmFK).

You may also want to [sign up for my newsletter](https://karlzylinski.substack.com/), where I summarize all my content month-by-month.

You can support my blog, my YouTube channel and my open-source projects by [becoming a patron](https://patreon.com/karl_zylinski/).

Have a nice day!
/Karl Zylinski