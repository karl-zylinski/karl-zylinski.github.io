---
title: "A common mistake: Dynamic array + arena"
date: 2025-04-04T12:29:00+02:00

draft: true

#cover:
  #image: "hba/cover.jpg"
---

A common mistake when working with arena allocators is to use the arena allocator as a backing allocator for a dynamic array. Let's look at why and what you can do instead.

> Arenas and arena allocators are useful for _grouping_ things that have the same lifetime. The same lifetime means that they should all be deallocated at the same time.

## A dynamic array with an arena allocator

Consider the following small program:

```go
package dynamic_array_mistake

import "core:fmt"
import "core:mem"
import "core:math/rand"

main :: proc() {
	arena_mem := make([]byte, 1*mem.Gigabyte)
	arena: mem.Arena
	mem.arena_init(&arena, arena_mem)
	fmt.println("Arena starts at address:", &arena_mem[0])

	arena_alloc := mem.arena_allocator(&arena)
	dyn_arr := make([dynamic]int, arena_alloc)
	append(&dyn_arr, 7)
	fmt.println("After 1 append to dynamic array, address of first element is:", &dyn_arr[0])

	for i in 0..<9999 {
		append(&dyn_arr, rand.int_max(100000))
	}

	fmt.println("After 10000 appends to dynamic array, address of first element is:", &dyn_arr[0])
}
```

There are several different kinds of arenas in Odin. Here we use the most simple one: The `mem.Arena`, it just needs a buffer of bytes to allocate into. So we allocate 1 Gigabyte of memory for it.

After initializing the arena with this block of memory we print the address of where the arena begins. On my computer it prints:
```txt
Arena starts at address: 0x17B04EEB048
```

We then create a dynamic array that can allocate into the arena. We append a single item into the dynamic array. If we print the address of the first element, this it says:

```txt
After 1 append to dynamic array, address of first element is: 0x17B04EEB048
```

That's the same as our arena. Sweet! So the dynamic array starts right at the beginning of the arena.

Now we add `9999` more items. That will make the dynamic array grow several times. After all that we print the address of the first element again:
```txt
After 10000 appends to dynamic array, address of first element is: 0x17B04F0AD48
```

That's not the beginning of the arena! How far away from the beginning is it? Subtract the two numbers top find out: `0x17B04F0AD48 - 0x17B04EEB048 = 0x1FD00`, or in decimal: `130304`. So the first element of the dynamic array is `130304` bytes from the start of the dynamic array.

## What happened?

![Shows how the memory usage of a dynamic array after 1 append and after 9 appends. After 1 append it sits at the start of the arena. After 9 appends it sits further away from the start and it has left the old data as a hole in the dynamic array](/arena-mistakes/growing_dyn_arr.png)

Initially the dynamic array has no memory allocated at all. After 1 append it will have allocated some initial memory. This initial memory will have space for 8 items. Imagine that 8 more items are then appended. This will make the dynamic array run out of capacity, forcing it to grow again. The new capacity will probably be something like 24.

When using a the default heap allocator this would mean that the dynamic array allocates more memory, then copies all its contents there and deallocates the old memory.

When it does the same using an arena allocator, then this happens:
- Tell arena allocator to allocate memory for new data block: OK! It gives you that many bytes (as long as it is not full).
- Copy old data to new data block: OK!
- Tell allocator to deallocate the old data: Nope! Arena allocators don't implement individual allocations. This step will have no effect.

So the old memory block is just left in the arena. Think of what happens if the dynamic array grows several times: You'll waste _a lot_ of memory. Using an arena as allocator for a dynamic array is a weird combination!

> Actually, the deallocation will return a `Mode_Not_Implemented` error. The dynamic array does not care about that error.

## Why can't the arena just deallocate the old block?

This kind of arena grows linearly. It just starts at the beginning of the arena. When allocations happen, it just gives out a pointer to the current position in the arena and moves forward by how many bytes you wanted to allocate. The next allocation will happen at the new position.

It's a simple allocator: It just knows where to take memory next and how much memory it has left. It does not keep track of all the previous allocations.

So individual deallocations can't happpen: It does not keep track of enough information to do that.

And even if it did track that information, doing deallocations in a linear block of memory like that would quickly lead to memory fragmentation: Imagine if you allocate using `new(Some_Type, arena_allocator)` and mix different types. Any deallocation would leave a hole of that size: Quite soon you'd have almost unusable holes in the memory. So it's better to keep it simple: Just a linearly growing block of memory.

## You've broken the core principle of arena allocators

Arena allocators are for things that share the same lifetime. This means that they should be deallocated at the same time.

In our example the dynamic array has its old block of memory in the arena as well as a new block. It tries to deallocate the old block but keep the new block: These two things clearly have different lifetimes!

This is another reason why individual allocations don't make any sense. Since everything in an arena should share the same life time, there is only one deallocation that makes sense: Deallocating everything in the arena. In our case, with the basic arena from `core:mem`, deallocating the arena means destroying the memory block:

```go
delete(arena_mem)
```
or, if you use something like a growing virtual arena from `core:mem/virtual`:

```
virtual.destroy_arena(&my_virtual_arena)
```

Now anything allocated into the arena is gone, in a single operation.

This is the whole point of the arena: You put things into it that should all be destroyed at the same time. It's not a silver bullet for making manual memory management "magically easy". It's just a way to group allocations so you don't have to do lots of separate `free` and `delete` calls in order to clean up.

## Growing virtual arena shenanigans

A very useful kind of arena is the growing virtual arena. It's arena that uses block of memory, when the current block is full, it allocates a new one and does allocations into that.

If you re-write our initial sample using a virtual growing arena, it would look like this:

```go
package dynamic_array_mistake_virtual

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

Wait a minute! It still has the same address after the 10000 appends! Is the growing virtual arena magical, does it support the individual deallocations that a dynamic array uses?! It seems to always sit at the same address.

No.

There's a special case in this arena that makes it reuse the same address if nothing else has been allocated in-between.

So if you had done any more allocation into the arena, then it would not be able to reuse the same address. It would have to move, and in so doing leave behind the derelict old data in the arena.

## Thanks for reading!

If you enjoyed this article, then perhaps you'd enjoy my book ["Understanding the Odin Programming Language"](https://odinbook.com). Read a free sample [here](https://odinbook.com/sample.html).

Ask questions on [my Discord server](https://discord.gg/4FsHgtBmFK).

You may also want to [sign up for my newsletter](https://karlzylinski.substack.com/), where I summarize all my content month-by-month.

You can support my blog, my YouTube channel and my open-source projects by [becoming a patron](https://patreon.com/karl_zylinski/).

Have a nice day!
/Karl Zylinski