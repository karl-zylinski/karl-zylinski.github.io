---
title: "Learn pointers by understanding how 'any' will ruin your day"
date: 2025-05-20T15:44:00+02:00

draft: true
---


In Odin there is a type called `any`. It has some specific use cases. But it is often used misused.

Internally, a variable of type `any` stores:
- A pointer.
- Information about what type the pointer refers to.

I'll get back to these pointer things in a bit. But lets first look at how things can go wrong when using `any`. I'll then dive into explaining why it it went wrong, and while doing so also explain how pointers and something known as _the stack_ works.

Below is a simple example of something that _seems_ to work fine.

```go
package any_gone_wrong

import "core:fmt"

thing: any

set_thing_to_7 :: proc() {
	thing = 7
}

main :: proc() {
	set_thing_to_7()
	fmt.println(thing) // 7
}
```

The code above has a global variable `thing` of type `any`. We set `thing` to the value `7` inside `set_thing_to_7` and then print the value of `thing` after `set_thing_to_7` has ended.

It will probably print `7`. So now you may think: "It successfully stored `7` inside `thing`... Oh, cool! `any` can store a value of any type. And it just works!"

Not really. Let's extend the example, which will cause things to break.

```go
package any_gone_wrong

import "core:fmt"

thing: any

set_thing_to_7 :: proc() {
	thing = 7
}

print_some_number :: proc() {
	something := 42
	fmt.println(something) // 42
}

main :: proc() {
	set_thing_to_7()
	fmt.println(thing) // 7
	print_some_number()
	fmt.println(thing) // 42
}
```
This example starts of the same as before: Set `thing` to `7` using the `set_thing_to_7` proc, and then print it.

After `set_thing_to_7` finishes, the program runs `print_some_number`. That one just makes a local variable with the number `42` in it, and then prints it. After `print_some_number` is done, we try printing `thing` again. On my computer the program outputs this:

```go
7
42
42
```

The third line is the result of `fmt.println(thing)`, the last line of `main`. Why is that one printing `42`? Why isn't it `7`?

The reason is that `any` does _not_ store a value. Rather, it stores a _pointer_.


I will get back to why it is good that `any` works like this, and what its primary use cases are. For now, let's first recreate the previous example, but without `any`, and instead use a global pointer. As we shall see, it will behave in the same way, and from understanding that behavior, we can understand pointers a bit better. And then we can understand `any`.

## maybe move this towards the end?

Below, the type of `thing` is instead `^int`, which is "pointer to integer".

```go
package pointer_gone_wrong

import "core:fmt"
import "core:math/rand"

thing: ^int

set_thing_to_7 :: proc() {
	stack_thing := 7
	thing = &stack_thing
}

print_random_number :: proc() {
	something := rand.int_max(100)
	fmt.println(something)
}

main :: proc() {
	set_thing_to_7()
	fmt.println(thing^) // 7
	print_random_number()
	fmt.println(thing^) // ???
}
```

The only other changes are that `set_thing_to_7` now creates a local variable `stack_thing` and then sets the value of `thing` so that it contains the _address of `stack_thing`_.

We also added `^` after thing when we call `fmt.println(thing^)`, so that it fetches the value that pointer refers to, instead of printing the pointer itself.

This version of our program will, when run, output something like this:

```
7
21
21
```

Again, the last print displays the same value as the print inside `print_random_number` did.

To understand what happened, we need to talk about two concepts: The stack and pointers.

Each procedure that you run has some memory that it can use for local variables. By local variables I meant variables such as `stack_thing`: It's a variable that lives for as long as that procedure runs. After that, it no longer exists. This is known as the _stack frame_ of the the procedure. It's the memory the proce