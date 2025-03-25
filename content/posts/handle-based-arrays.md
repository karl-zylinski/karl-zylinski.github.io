---
title: "\"Handles are the better pointers\": An Odin gamedev follow-up"
date: 2025-03-25T12:59:00+01:00

cover:
  image: "hba/cover.jpg"
---

## Background
Andre Weissflog wrote an article in 2018 called ["Handles are the better pointers"](https://floooh.github.io/2018/06/17/handles-vs-pointers.html). In it he talks about how one can use _handles_ instead of _pointers_ when one wants to store permanent references to elements in an array.

I think there are many nuances regarding handles that only become apparent after some real-world usage. Here we'll look at an implementation in Odin and discuss it.

> I use the [Odin Programming Language](https://odin-lang.org) and the examples are game development-centric.

## What's a handle?

A handle is often just an _index_ and a _generation_. You can store that in a struct:

```go
Handle :: struct {
	index: u32,
	generation: u32,
}
```

These fields are `32` bits each. Exactly how big you make them depends on your needs. An `u32` for the index means you can have up to `4294967295` items in the array.

The `index` field is the index of an array element. The `generation` is used to make sure that the thing at `index` is still the same thing as when the handle was acquired.

## An example implementation

Let me dump an example implementation here. Then I'll discuss what it does and how to use it. After that we'll discuss how you can improve it.

> Use this code by copying this file into your program's package and changing the `package game` line.

> This example is also available as a [gist](https://gist.github.com/karl-zylinski/a5c6acd551473f90b872f46a2fa58deb).

```go
// Handle-based array. Used like this:
// 
// Entity_Handle :: distinct Handle
// entities: Handle_Array(Entity, Entity_Handle)
//
// It expects the type used (in this case
// Entity) to contains a field called
// `handle`, of type `Entity_Handle`.
//
// You can then fetch entities using
// ha_get() and add handles using ha_add().
//
// This is just an example implementation.
// There are many ways to implement this.
//
// Another example by Bill:
// https://gist.github.com/gingerBill/7282ff54744838c52cc80c559f697051
//
// Bill's example is a bit more
// complicated, but a bit more efficient.
// It doesn't have to loop over all items
// to "skip unoccupied holes".
//
// Jakub also has an implementation of
// something similar. But his version
// doesn't use dynamic memory:
// https://github.com/jakubtomsu/sds/blob/main/pool.odin

package game

Handle :: struct {
	idx: u32,
	gen: u32,
}

HANDLE_NONE :: Handle {}

Handle_Array :: struct($T: typeid, $HT: typeid) {
	items: [dynamic]T,
	freelist: [dynamic]HT,
	num: int,
}

ha_clear :: proc(ha: ^Handle_Array($T, $HT)) {
	clear(&ha.items)
	clear(&ha.freelist)
}

ha_delete :: proc(ha: Handle_Array($T, $HT)) {
	delete(ha.items)
	delete(ha.freelist)
}

ha_add :: proc(ha: ^Handle_Array($T, $HT), v: T) -> HT {
	v := v

	if len(ha.freelist) > 0 {
		h := pop(&ha.freelist)
		h.gen += 1
		v.handle = h
		ha.items[h.idx] = v
		ha.num += 1
		return h
	}

	if len(ha.items) == 0 {
		append_nothing(&ha.items) // Item at index zero is always "dummy" used for zero comparison
	}

	idx := u32(len(ha.items))
	v.handle.idx = idx
	v.handle.gen = 1
	append(&ha.items, v)
	ha.num += 1
	return v.handle
}

ha_get :: proc(ha: Handle_Array($T, $HT), h: HT) -> (T, bool) {
	if h.idx > 0 && int(h.idx) < len(ha.items) && ha.items[h.idx].handle == h {
		return ha.items[h.idx], true
	}

	return {}, false
}

ha_get_ptr :: proc(ha: Handle_Array($T, $HT), h: HT) -> ^T {
	if h.idx > 0 && int(h.idx) < len(ha.items) && ha.items[h.idx].handle == h {
		return &ha.items[h.idx]
	}

	return nil
}

ha_remove :: proc(ha: ^Handle_Array($T, $HT), h: HT) {
	if h.idx > 0 && int(h.idx) < len(ha.items) && ha.items[h.idx].handle == h {
		append(&ha.freelist, h)
		ha.items[h.idx] = {}
		ha.num -= 1
	}
}

ha_valid :: proc(ha: Handle_Array($T, $HT), h: HT) -> bool {
	return ha_get(ha, h) != nil
}

// Iterators for iterating over all used
// slots in the array. Used like this:
//
// ent_iter := ha_make_iter(your_handle_based_array)
// for e in ha_iter_ptr(&ent_iter) {
//
// }

Handle_Array_Iter :: struct($T: typeid, $HT: typeid) {
	ha: Handle_Array(T, HT),
	index: int,
}

ha_make_iter :: proc(ha: Handle_Array($T, $HT)) -> Handle_Array_Iter(T, HT) {
	return Handle_Array_Iter(T, HT) { ha = ha }
}

ha_iter :: proc(it: ^Handle_Array_Iter($T, $HT)) -> (val: T, h: HT, cond: bool) {
	in_range := it.index < len(it.ha.items)

	for in_range {
		cond = it.index > 0 && in_range && it.ha.items[it.index].handle.idx > 0

		if cond {
			val = it.ha.items[it.index]
			h = it.ha.items[it.index].handle
			it.index += 1
			return
		}

		it.index += 1
		in_range = it.index < len(it.ha.items)
	}

	return
}

ha_iter_ptr :: proc(it: ^Handle_Array_Iter($T, $HT)) -> (val: ^T, h: HT, cond: bool) {
	in_range := it.index < len(it.ha.items)

	for in_range {
		cond = it.index > 0 && in_range && it.ha.items[it.index].handle.idx > 0

		if cond {
			val = &it.ha.items[it.index]
			h = it.ha.items[it.index].handle
			it.index += 1
			return
		}

		it.index += 1
		in_range = it.index < len(it.ha.items)
	}

	return
}
```

## Creating an array

Create a handle-based array like this:

```go
Entity :: struct {
	my_data: int,
}

Entity_Handle :: distinct Handle
entities: Handle_Array(Entity, Entity_Handle)
```

> Here `Entity` means an object in a game world.

We define a distinct type `Entity_Handle` for our handles. This makes the handles type safe. We don't want to accidentally mix up different kinds of handles.

## Adding and getting items

Add to the array like so:
```go
h := ha_add(&entities, Entity { my_data = 7 })
```

`ha_add` returns a handle of type `Entity_Handle`. We can store that handle in order to keep a reference to this new entity.

Later, you can fetch a pointer using the handle:

```go
if e := ha_get_ptr(&entities, h); e != nil {
	e.my_data = 42
}
```

Here we use `ha_get_ptr` which will use the handle to fetch a pointer to the thing it refers to.

**IMPORTANT**: Only fetch the pointer when you need it! The whole point is that you store handles permanently (between frames) and then temporarily resolve the handle into a pointer.

Looking at what `ha_get_ptr` does, we can see some of the smart properties of a handle-based array:

```go
ha_get_ptr :: proc(ha: Handle_Array($T, $HT), h: HT) -> ^T {
	if h.idx > 0 && int(h.idx) < len(ha.items) && ha.items[h.idx].handle == h {
		return &ha.items[h.idx]
	}

	return nil
}
```

The if-statement first checks that the index inside the handle is within the range of the handle-based array. But then it also checks this:
```go
ha.items[h.idx].handle == h
```
Remember: The handle contains both the index _and_ the generation. The handle `h` and the `handle` field of the element will only be equal if both the index and the generation match. This makes sure that the thing at that index _has not been replaced_. This is why the generation is so important: It makes it possible to store a handle and be sure that it still refers to the same thing that it initially did.

There's also a `ha_get` procedure that returns a value instead of a pointer:

```go
if e, e_ok := ha_get(entities, h); e_ok {
	fmt.println(e.my_data)
}
```

Note how `ha_get` returns a second value of type `bool`. It's used to say if a valid item was found or not. 

## Iterating the handle-based array

The implementation provides procedures to iterate over the array:

```go
ent_iter := ha_make_iter(&entities)
for e in ha_iter_ptr(&ent_iter) {
	fmt.println(e.my_data)
}
```

Why is a custom iterator required? Why can't you just do `for e in entities.items {}`? Because `items` may contain unused items. If anything has been removed from the array, then that item is just marked as unused. It is like that in order to keep the indices stable. This iterator will skip those "holes".

> An unused item is defined as it having `handle.idx == 0`. The zero index is never used. You can see how `ha_add` always adds a "dummy element" at index 0. This is good for zero-is-initialized programming: Handles can be zero initialized without accidentally referring to the first item in an array.

> `ha_iter_ptr` does the iteration by looping over all the items and skipping the unused ones. This can probably be done in a more efficient way by creating some kind of linked list where each item knows where the next valid item is. But it hasn't been a big deal for my use cases, so it might be OK for you too. I suggest not doing anything about it unless you profile and see too much time spent iterating handle-based arrays.

## Discussion: Problems with dangling _temporary_ pointers

While the handles are very stable references, things can still go wrong.

For example, you might fetch a pointer using a handle. Then you call some code that may grow the array. Then you use the pointer, which may now be dangling! Like this:

```
e := ha_get_ptr(&entities, h)

ha_add(&entities, Entity { my_data = 123 })

if e != nil {
	fmt.println(e.my_data)
}
```

The `ha_add` call may grow the array and thus make the pointer `e` invalid!

Now you may shout: "But that's exactly what you shouldn't do!! Just fetch the item using the handle when you need it, and use it!"

I can promise you, that if you spend some time working on for example a big game, then you'll run into cases where this happens.

For example:
```
e := ha_get_ptr(&entities, h)

do_cool_thing(e)

if e != nil {
	fmt.println(e.my_data)
}
```

What does this code do? It seems to do a cool thing with an entity pointer. Let's say you need `do_cool_thing` to take a pointer because it needs to modify the entity. But 3 months later maybe you add some code to `do_cool_thing` that adds a temporary entity, such as a graphical effect. That adding of an entity may grow the handle-based array... and... uh-oh! Dangling pointer!

Initially you may think that you can avoid this by making sure that no code that adds anything to the array is called while you're doing stuff with a pointer. But when you are trying to ship something, then you'll start adding polish. Polish means adding lots of tiny extra things. Such as effects. And you'll add them in the most convenient places, so that you don't break any other code. You _will_ end up with dangling pointers sooner or later.

### Solution 1: Fixed-size array

In my game [CAT & ONION](https://zylinski.itch.io/cat-and-onion) I worked around this by changing the implementation of `Handle_Array` to use a fixed-size dynamic array. It is not allowed to grow. I just estimate a good upper boundary for it. This means that adding things to it will never create dangling pointers. It's always at maximum capacity!

You can do that by either using actual fixed arrays, or by allocating a dynamic array once and setting its allocator to the panic allocator. The latter is what CAT & ONION does:

```
ha_add :: proc(ha: ^Handle_Array($T, $HT), v: T) -> HT {
	if ha.items == nil {
		ha.allocator = context.allocator
		ha.items = make([dynamic]T, 0, HANDLE_ARRAY_SIZE)
		ha.items.allocator = mem.panic_allocator()
		ha.unused_items = make([dynamic]u32)
	}

	// ha_add as usual
```

You could also move `HANDLE_ARRAY_SIZE` into the `Handle_Array` here, in case you need `Handle_Array`s of different types.

Now you may go: "But if the array is of fixed size, why do you then need handles?! It can't grow, no more dangling pointers. Throw the handles over board!!"

Sure, you can now store a pointer to an array item. But how can you be sure that it is still the same item 10 minutes later? You still need the handle because it contains the _generation_, and the generation makes sure that things are not replaced under your nose.

### Solution 2: Grow array when its safe

You could also add the newly added things to a separate array, and only _commit_ those things into the main array at a "safe spot" in your code.

I did some experiments with this. You can see those experiments [here](https://gist.github.com/karl-zylinski/68efd204eeb46ef813538ac35a4c13ac). But here's the whole thing, for posterity in case GitHub has a sad day:

```go
package game

import "core:fmt"
import "core:mem"
import "core:slice"
import vmem "core:mem/virtual"

_ :: fmt

Handle :: struct {
	idx: u32,
	gen: u32,
}

HANDLE_NONE :: Handle {}

Handle_Array :: struct($T: typeid, $HT: typeid) {
	items: [dynamic]T,
	unused_items: [dynamic]u32,
	allocator: mem.Allocator,

	// To make sure we do not invalidate `items` in the middle of a frame, while there are pointers
	// to things it, we add new items to this, and the new items are allocated using the growing
	// virtual arena `new_items_arena`. Run `ha_commit_new` once in a while to move things from
	// `new_items` to `items`.
	new_items: [dynamic]^T,
	new_items_arena: vmem.Arena,
}

ha_delete :: proc(ha: Handle_Array($T, $HT), loc := #caller_location) {
	delete(ha.items, loc)
	delete(ha.unused_items, loc)
}

ha_clear :: proc(ha: ^Handle_Array($T, $HT), loc := #caller_location) {
	clear(&ha.items)
	clear(&ha.unused_items)
	clear(&ha.new_items)
}

// Call this at a safe space when there are no pointers in flight. It will move things from
// new_items into items, potentially making it grow. Those new items live on a growing virtual
// memory arena until this is called.
ha_commit_new :: proc(ha: ^Handle_Array($T, $HT), loc := #caller_location) {
	if len(ha.items) == 0 {
		// Dummy item at idx zero
		append(&ha.items, T{})
	}

	for ni in ha.new_items {
		if ni == nil {
			// We must add these, if we don't the indices get out of order with regards to handles
			// we have handed out. We'll just add them as empty objects and then put the index into
			// unused items.
			unused_item_idx := len(ha.items)
			append(&ha.items, T {
				handle = {
					gen = 1,
				},
			})

			append(&ha.unused_items, u32(unused_item_idx))
			continue
		}

		append(&ha.items, ni^)
	}

	vmem.arena_free_all(&ha.new_items_arena)
	ha.new_items = {}
}

ha_clone :: proc(ha: Handle_Array($T, $HT), allocator := context.allocator, loc := #caller_location) -> Handle_Array(T, HT) {
	return Handle_Array(T, HT) {
		items = slice.clone_to_dynamic(ha.items[:], allocator, loc),
		unused_items = slice.clone_to_dynamic(ha.unused_items[:], allocator, loc),
		allocator = allocator,
	}
}

ha_add :: proc(ha: ^Handle_Array($T, $HT), v: T, loc := #caller_location) -> HT {
	if ha.items == nil {
		if ha.allocator == {} {
			ha.allocator = context.allocator
		}

		ha.items = make([dynamic]T, ha.allocator, loc)
		ha.unused_items = make([dynamic]u32, ha.allocator, loc)
	}

	v := v

	if len(ha.unused_items) > 0 {
		reuse_idx := pop(&ha.unused_items)
		reused := &ha.items[reuse_idx]
		h := reused.handle
		reused^ = v
		reused.handle.idx = u32(reuse_idx)
		reused.handle.gen = h.gen + 1
		return reused.handle
	}

	if len(ha.items) == 0 {
		// Dummy item at idx zero
		append(&ha.items, T{})
	}

	new_items_allocator := vmem.arena_allocator(&ha.new_items_arena)
	new_item := new(T, new_items_allocator)
	new_item^ = v
	new_item.handle.idx = u32(len(ha.items) + len(ha.new_items))
	new_item.handle.gen = 1

	if ha.new_items == nil {
		ha.new_items = make([dynamic]^T, new_items_allocator)
	}

	append(&ha.new_items, new_item)
	return new_item.handle
}

ha_get :: proc(ha: Handle_Array($T, $HT), h: HT) -> (T, bool) #optional_ok {
	if ptr := ha_get_ptr(ha, h); ptr != nil {
		return ptr^, true
	}

	return {}, false
}

ha_get_ptr :: proc(ha: Handle_Array($T, $HT), h: HT) -> ^T {
	if h.idx == 0 || h.idx < 0 {
		return nil
	}

	if int(h.idx) >= len(ha.items) {
		// The item we look for might be in `new_items`, so look in there too
		new_idx := h.idx - u32(len(ha.items))
		
		if new_idx >= u32(len(ha.new_items)) {
			return nil
		}

		if item := ha.new_items[new_idx]; item != nil && item.handle == h {
			return item
		}

		return nil
	}

	if item := &ha.items[h.idx]; item.handle == h {
		return item
	}

	return nil
}

ha_remove :: proc(ha: ^Handle_Array($T, $HT), h: HT) {
	if h.idx == 0 || h.idx < 0 {
		return
	}

	if int(h.idx) >= len(ha.items) {
		new_idx := h.idx - u32(len(ha.items))
		
		if new_idx < u32(len(ha.new_items)) {
			// This stops this item from being added during `ha_commit_new`
			ha.new_items[new_idx] = nil
		}

		return
	}

	if item := &ha.items[h.idx]; item.handle == h {
		append(&ha.unused_items, h.idx)

		// This makes the item invalid. We'll set the index back if the slot is reused.
		item.handle.idx = 0
	}
}

ha_valid :: proc(ha: Handle_Array($T, $HT), h: HT) -> bool {
	return ha_get_ptr(ha, h) != nil
}

Handle_Array_Iter :: struct($T: typeid, $HT: typeid) {
	ha: ^Handle_Array(T, HT),
	index: int,
}

ha_make_iter :: proc(ha: ^Handle_Array($T, $HT)) -> Handle_Array_Iter(T, HT) {
	return { ha = ha }
}

ha_iter :: proc(it: ^Handle_Array_Iter($T, $HT)) -> (val: T, h: HT, cond: bool) {
	val_ptr: ^T

	val_ptr, h, cond = ha_iter_ptr(it)

	if val_ptr != nil {
		val = val_ptr^
	}

	return
}

ha_iter_ptr :: proc(it: ^Handle_Array_Iter($T, $HT)) -> (val: ^T, h: HT, cond: bool) {
	cond = it.index < len(it.ha.items) + len(it.ha.new_items)

	for ; cond; cond = it.index < len(it.ha.items) + len(it.ha.new_items) {
		// Handle items in new_items
		if it.index >= len(it.ha.items) {
			idx := it.index - len(it.ha.items)

			if it.ha.new_items[idx] == nil {
				it.index += 1
				continue
			}

			val = it.ha.new_items[idx]
			h = val.handle
			it.index += 1
			break
		}

		if it.ha.items[it.index].handle.idx == 0 {
			it.index += 1
			continue
		}

		val = &it.ha.items[it.index]
		h = val.handle
		it.index += 1
		break
	}

	return
}

// Test handle array and basic usage documentation.

ha_test :: proc() {
	Ha_Test_Entity :: struct {
		handle: Ha_Test_Entity_Handle,
		pos: [2]f32,
		vel: [2]f32,
	}

	Ha_Test_Entity_Handle :: distinct Handle

	ha: Handle_Array(Ha_Test_Entity, Ha_Test_Entity_Handle)

	h1 := ha_add(&ha, Ha_Test_Entity {pos = {1, 2}})
	h2 := ha_add(&ha, Ha_Test_Entity {pos = {2, 2}})
	h3 := ha_add(&ha, Ha_Test_Entity {pos = {3, 2}})

	ha_remove(&ha, h2)

	// This one will reuse the slot h2 had
	h4 := ha_add(&ha, Ha_Test_Entity {pos = {4, 2}})
	assert(h4.idx == 4)

	assert(h1.idx == 1)
	assert(h2.idx == 2)
	assert(h3.idx == 3)
	assert(h1.gen == 1)
	assert(h2.gen == 1)
	assert(h3.gen == 1)

	if _, ok := ha_get(ha, h2); ok {
		panic("h2 should not be valid")
	}

	if h4_ptr := ha_get_ptr(ha, h4); h4_ptr != nil {
		assert(h4_ptr.pos == {4, 2})
		h4_ptr.pos = {5, 2}
	} else {
		panic("h4 should be valid")
	}

	if h4_val, ok := ha_get(ha, h4); ok {
		assert(h4_val.pos == {5, 2})
	} else {
		panic("h4 should be valid")
	}

	// This call moves new items from new_items into items. Needs to be run for example at
	// end of frame in a game.
	ha_commit_new(&ha)

	if h4_val, ok := ha_get(ha, h4); ok {
		assert(h4_val.pos == {5, 2})
	} else {
		panic("h4 should be valid")
	}

	ha_remove(&ha, h4)
	h5 := ha_add(&ha, Ha_Test_Entity {pos = {6, 2}})
	assert(h5.idx == 4)

	ha_delete(ha)
}
```

Look, it even comes with a `test` proc that you can use to see how it works! The thing to note here is that it calls `ha_commit_new` when it wants to move new things into the main array. You could do that at end of the frame, when nothing more will happen and all pointers-resolved-from-handles have gone to bed.

This implementation even comes with an iterator that makes sure to include the newly added things. The newly added things live in a separate array. Each new item is separately allocated into a growing virtual arena. See these lines in the `Handle_Array` struct:
```go
/*
To make sure we do not invalidate `items`
in the middle of a frame, while there are
pointers to things it, we add new items to
`new_items`, and the new items are allocated
using the growing virtual arena
`new_items_arena`. Run `ha_commit_new` once
in a while to move things from `new_items`
to `items`.
*/
new_items: [dynamic]^T,
new_items_arena: vmem.Arena,
```

We separately allocate the new items using an arena because if you did `new_items: [dynamic]T`, then some pointer to a newly-added-by-not-yet-committed item may become invalid if `new_items` grow. Which is... the problem we tried to solve for the main `items` array.

Now you may have galaxy brain moment: "Wait, if using the virtual growing arena makes things not move in memory, couldn't I use it for everything in the handle-based array?"

Probably! But that is further than I have experimented.

You could maybe remove the `new_items` stuff from `Handle_Array` and instead do this:

```go
items: [dynamic]^T
items_arena: vmem.Arena
```

And then change `ha_add` to add items to the arena using that allocator, like so:

```go
items_allocator := vmem.arena_allocator(&ha.items_arena)
item := new(T, new_items_allocator)
item^ = v
append(&ha.items, item)
```

Even though `items` consists of pointers, all those pointers actually point to things that live one-next-to-each-other in memory, thanks to the nature of the arena allocator.

But like I said, I have not tried this, but perhaps it's a nice way for you.

## This got complicated! What are your recommendations?

I know, that's why I wrote this. These things are simple at first, but can turn complicated quickly.

Since this may be an important foundational container in your game, I recommend you keep it simple. Perhaps the fixed-size array approach is a good start. A fixed-size array plus a generation makes sure that pointers are stable and things still refer to the thing we thought it did.

## Further examples

Jakub has made a something similar that uses true fixed arrays: https://github.com/jakubtomsu/sds/blob/main/pool.odin

Bill wrote an example of this that has many similarities to what I showed here: https://gist.github.com/gingerBill/7282ff54744838c52cc80c559f697051

## Thanks for reading!

If you enjoyed this article, then perhaps you'd enjoy my book ["Understanding the Odin Programming Language"](https://odinbook.com). Read a free sample [here](https://odinbook.com/sample.html).

You may also want to [sign up for my newsletter](https://karlzylinski.substack.com/), where I summarize all my content month-by-month.

You can support my blog, my YouTube channel and my open-source projects by [becoming a patron](https://patreon.com/karl_zylinski/).

Have a nice day!
/Karl Zylinski