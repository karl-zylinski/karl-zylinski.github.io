---
title: "Handle-based arrays in Odin"
date: 2025-03-25T12:59:00+01:00

draft: true
#cover:
#  image: "writing_book/cover.png"
---

## Problem: Dangling pointers to array elements

It is often a good idea to store array elements directly in the array, with no indirection. Something like this:
```
entities: [dynamic]Entity
```
> Some people would use `entities: [dynamic]^Entity` and separately heap allocate each entity (using `new(Entity)`). I explain why this is a bad idea in my blog post ["Data-oriented ideas for small gamedev teams"](https://zylinski.se/posts/data-oriented-ideas-for-small-gamedev-teams/).

However, this dynamic array might grow. When a dynamic array grows all its data may get moved to a new place in memory. This means that any pointer you've stored to an array element now points at garbage data.

Here's an example of how it goes wrong:

```
Entity :: struct {
	position: [3]f32
}

entities: [dynamic]Entity

for i in 0..<5 {
	append(&entities, Entity{})
}

entity_ptr := &entities[2]

for i in 0..<10000 {
	append(&entities, Entity{})
	entity_ptr_cur := &entities[2]
	if entity_ptr != entity_ptr_cur {
		fmt.panicf("After %i calls to append: Address of entity_ptr (%p) is not same as &entities[2] (%p)", i, entity_ptr, entity_ptr_cur)
	}
}
```

Running this program will probably print something like:
```
panic: After 115 calls to append: Address of entity_ptr (0x1ABA33E8FB0) is not same as &entities[2] (0x1ABA33ECCB0)
```
The number `115` will be different on each computer.

What the program does is save the address of the third element of the dynamic array `entities`. It then loops 10000 times and adds one more element during each lap. For each lap of the loop, it checks if the address of the third element of the array is still the same as the one it saved away before the loop.

Soon or later they will not be the same, due to the dynamic array having grown and moved its data.
> A dynamic array sometimes doesn't move its data when growing, if it could grow in-place. Sometimes there just isn't memory in that region enough to grow, which forces it to move.

Using `entity_ptr` after this would result in bugs and crashes. That pointer looks at where ever the old data of the dynamic array lived.

## A basic handle: It's just an index!

A way to work around this is to store the index of the element instead of a pointer:

```
Entity :: struct {
	position: [3]f32
}

entities: [dynamic]Entity

for i in 0..<5 {
	append(&entities, Entity{})
}

entity_idx := 2

for i in 0..<10000 {
	append(&entities, Entity{})
}

entity_ptr := &entities[entity_idx]
```

Note how we fetch `entity_ptr` using the index `entity_idx` after adding all those `10000` items. No matter how many times the array's data moved, you'd still get a pointer to the third element using the index we saved away earlier.

This index is our first rudimentary _handle_: You can store this index inside any struct to refer back to that item. Unlike storing a pointer it will never become stale because the array grew.

## Problem: Array shrinks

If the array shrinks, then your index may refer to something outside the array:
```
Entity :: struct {
	position: [3]f32
}

entities: [dynamic]Entity

for i in 0..<5 {
	append(&entities, Entity{})
}

entity_idx := 2
resize(&entities, 1)

// Bad pointer!
entity_ptr := &entities[entity_idx]
```

The code above has added a `resize(&entities, 1)`. This makes our index `2` fall outside the array. Running this code would result the following error:
```
Index 2 is out of range 0..<1
```

To avoid this we must check if the index is in range. We can do that using a procedure:

```
entity_get :: proc(arr: ^[dynamic]Entity, idx: int) -> ^Entity {
	if idx < 0 || idx >= len(arr) {
		return nil
	}

	return &arr[idx]
}

entity_ptr := entity_get(&entities, entity_idx)
```

Now you'd get `nil` back for any index outside of `entities`, instead of crashing. Make sure to check for nil before using it!




You may have used pointers to array elements in your code, only to end up with dangling pointers and crashes.