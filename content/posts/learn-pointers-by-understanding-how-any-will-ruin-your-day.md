---
title: "Learn pointers by understanding how 'any' will ruin your day"
date: 2025-05-20T15:44:00+02:00

draft: true
---


In Odin there is a type called `any`. It has some specific use cases. But it is often used misused. It can store a reference to a value of a generic type.

Below is an example of something that may _seem_ like it works fine, but at we shall see it doesn't.

```go
package any_gone_wrong

import "core:fmt"

thing: any

main :: proc() {
	set_thing_to_7()
	fmt.println(thing)
}

set_thing_to_7 :: proc() {
	a_number: int = 7
	thing = a_number
}
```

If you save the file above in a folder, navigate to that folder and execute:

```
odin run .
```

Then the program will probably print `7`. So now you may think: "It successfully stored `7` inside `thing`... Oh, cool! `any` can store a value of any type. And it just works!"

Not so fast.

Try doing this instead:

```
odin run . -o:speed
```

On my machine, it no longer prints `7`, but instead `0`. What `-o:speed` does is optimize the program so that it runs faster.

What happened here? In order to explain this, let's look at what `any` actually is.

All variables of type `any` look like this "under the hood":

```go
Raw_Any :: struct {
	data: rawptr,
	id:   typeid,
}
```
> You'll find this struct in `<odin_dir>/base/runtime/core.odin`.

The following code
```go
a_number: int = 7
thing = a_number
```
is actually the same as this:
```go
a_number: int = 7
thing.data = &a_number
thing.id = type_of(a_number)
```

Note the `&`. That's the "address of" operator. That operator fetches the memory address of `a_number`. So the any does not contain any value at all! It just contains the memory address of something.

Let's visualize what this address actually is. The follwing procedure has just a single local variable: `a_number`. That local variable must live somewhere. There must be a place in the computer's memory where it stores `a_number`'s value.
```go
set_thing_to_7 :: proc() {
	a_number: int = 7
	thing = a_number
}
```

When a procedure is executed, it is handed some memory called a _stack frame_. Inside this memory there is room for the procedure's local variables. The stack frame of `set_thing_to_7` is put "on top of" the stack frame for `main`. So these stack frames make up what is known as _the stack_. In other words, the address of `a_number` is a number that says at what position in memory the value of `a_number` lives. This address is within the stack frame.

When a procedure like `set_thing_to_7` ends, then you can no longer expect anything that lives within its stack frame to be valid. Whenever some other procedure is called, that memory will be repurposed for that procedure.

So since `thing = a_number` implictly fetches the address of `a_number` (`thing.data = &a_number`), then `thing` points at invalid stack memory after `set_thing_to_7` finishes.

So when `fmt.println(thing)` is run in `main`, then it shows us the value `7` by going through the `data` pointer of `thing` and to the old, discarded stack memory of `set_thing_to_7`.

Note that calling `fmt.println(thing)` sets up another stack frame for the `println` procedure. It will end up in the same memory region as `set_thing_to_7`, but we are just lucky that the `7` isn't replaced by anything else.

This bring us back to

```
odin run . -o:speed
```

Running the program like this did not print `7`. But curiously, if you add `fmt.println("hellope!)` into `set_thing_to_7`, then it will print `7`, even with `-o:speed`:

```
package any_gone_wrong

import "core:fmt"

thing: any

main :: proc() {
	set_thing_to_7()
	fmt.println(thing)
}

set_thing_to_7 :: proc() {
	a_number: int = 7
	thing = a_number
	fmt.println("hellope!")
}
```

What `-o:speed` does, is among other things, to remove unused variables and parameters. There is an implicit `context` parameter passed everywhere in Odin. `set_thing_to_7` did not use `context` without the `fmt.println("hellope!)`: `-o:speed` removed it. This causes `a_number` to instead live in the region where the `context` would have lived.

So without that extra "hellope!", when `fmt.println(thing)` is called, it will set up a new stack frame. That stack frame will need the context and that parameter will then overwrite the 

 When we put `fmt.println("hellope!")` inside `set_thing_to_7`, then the `context` is needed, so it can no longer be optimized away.