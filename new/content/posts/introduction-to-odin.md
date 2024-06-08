---
title: "Introduction to the Odin Programming Language"
date: 2024-06-07T21:16:00+02:00

cover:
  image: "/odin-intro.png"
---

## Preface

This article is an introduction the Odin Programming Language. It is aimed at people who know a bit of programming, but have never touched Odin. It is meant to be read from start to finish. It is not a reference guide, but will rather give you a feeling for what the language does differently and what you can expect as you venture forth. There will be some notes on differences to C/C++, as Odin in many ways tries to be better C. If you enjoy this article and want to support me, then you can do so by [becoming a patron](https://www.patreon.com/karl_zylinski).

In the recent years most of my programming has been done using the Odin Programming Language, sometimes referred to as Odinlang. Since a year back I create my own video games using Odinlang, you can find my game [CAT & ONION](https://store.steampowered.com/app/2781210/CAT__ONION/) on Steam, it is completely created in Odin.

Why Odin? I come from having worked on video game engines where I programmed a lot in C++ and C. While those languages are very powerful, they are also very old and have a lot of baggage. Odin strives to be a modern alternative to C, it doesn't try to fundamentally solve any new problems. It is not a Big Agenda language, meaning that it doesn't try to enforce a new way of programming onto you.

When I first started programming Odin it felt like it point-for-point fixed many of the issues I had with C. Instead of listing those issues here, I will point out those differences throughout this article.

Another reason for "Why Odin?" is that I prefer programming languages that fulfill these three properties:
- Compiled (not interpreted)
- Manual memory management
- Statically typed

Many new languages fail on one of these points, so I end up not liking them. However, the last few years there have been some popular languages that fit into this description, such as Rust and Zig. If you want to use one of Odin, Rust or Zig but don't know which one, then I recommend you to spend 1-2 weeks using each and make up your mind. Do not listen to fans of these languages, because a lot of it boils down to personal preference.

## A note on learning Odin

While Odin is a great language, the area it is currently most lacking in is documentation. There is lots of documentation, but it is scattered and incomplete. I tried to address that by writing this article, which I hope gives a holistic introduction to Odin. This article aside, here is my recommendation of where to look if you are stuck on trying to figure out how to do something specific in Odin. I recommend you check these resources in the order stated:

1. Check the [overview](https://odin-lang.org/docs/overview/). The overview is not complete, but searching around in there will answer many beginner questions.
2. Look inside [demo.odin](https://github.com/odin-lang/Odin/blob/master/examples/demo/demo.odin). It is an example that comes with the compiler that tries to showcase many different features of the language.
3. Search in the core library. I.e. pull `<your_odin_compiler_dir>/core` into your text editor and just search around in there. The core library is well written and easy to read, it is a great way to learn many Odin concepts.
4. Ask in the #beginners channel on the [Odin Discord server](https://discord.com/invite/sVBPHEv). The server has thousands of members and many questions get answered very quickly.

## Installing Odin

I will not go over in detail how to install Odin. You can read about that on the [Getting Started](https://odin-lang.org/docs/install/) page. If you still have trouble, then I have a video on how to setup Odin from source:

{{<youtube yq5VabsGz_4>}}

## Basic "Hello World"

Here is a tiny Hello World program written in Odin:
```C
package hello_world

import "core:fmt"

main :: proc() {
	fmt.println("Hello World!")
} 
```

Functions are in Odin referred to as procedures. `main` is the default entry point for the program. We see that procedures are created by typing `procedure_name :: proc() {}`

The line `import "core:fmt"` fetches the `fmt` package from the standard library. `fmt` contains procedures that can print to the console and also format strings. All those things inside the `fmt` package can then be used by typing `fmt.some_procedure_name()`. Inside the `main` proc we use the `fmt.println` proc to print "Hello World" to the console.

Semicolons are not needed, and I would encourage to simply not use them. Not using semicolons fits well with how Odin looks. You can still use them if you really want to! There is also a compiler flag to make it an error if you put in unnecessary semicolons (`-vet-semicolon`)

If you copy-paste the code above into a file and save that file inside a new folder, then you can run the program by navigating to that folder and running: `odin run .`

`.` refers to the current directory. This means that it will take all the things in the current directory and compile them as a single package, output an executable and run it.

I will say more about the package system later, but for now I just want to point out that the line `package hello_world` must be the same for all odin files within a folder.

> The reason for saying procedure, or proc, instead of function is that the word function, if one wants to be nit-picky, is a bunch of code that has _no side effects_. This is why when we talk about "functional programming" we mean code that composes new functionality from combining functions, where those functions do not modify any global state. Now, functions can be seen as a special case of procedures. I.e. a procedure can take some parameters, return some values and also have side-effects on global variables, while functions can do the same except for that they have no side-effects. Some languages refer to side-effect free procedures as "pure functions".

## Variables and loops

Let's add some variables and a loop to our Hello World program:

```C
package hello_world

import "core:fmt"

main :: proc() {
	a_string := "Hello World!"
	fmt.println(a_string)

	counter: int

	for counter < 20 {
		fmt.printfln("Counter value: %v", counter)
		counter += 1
	}
} 
```

The line `a_string := "Hello World!"` declares and initializes a variable. Odin is a strongly typed language, meaning that all variables must have a specific type. But the type can be inferred from the right-hand side, which happens here. In this case `a_string` will be of type `string`.

The line `counter: int` also declares a variable, but this time it only says which type, but no value is given. Since no value is given, it will be initialized to zero. If you for some reason do not want to have it initialized, then you can write `counter: int = ---`. However, in almost all cases you will want zero initialization. In some performance sensitive algorithms you may want to skip it, but I have almost never had to do so.

The language and the core library (Odin's standard library) also tries to use the zero value as a 'good default' as much as possible, which is known as zero-is-initialized. Zero-is-initialized and automatic zero initialization is a comfy combo.

> In C and C++ if you write `int counter;` then the variable counter will not be initialized and can just contain garbage memory, which is almost never what you want.

Let's look close at `:` and `=`, in particular how the usage of these characters vary when declaring and setting variables:

```C
a_string := "Hello World!"
counter: int
a_decimal_number: f32 = 2
a_decimal_number = 3.2
```

Only having a `=`, like the last line, is only valid if there already exists a variable with that name. You can also force a variable to have specific type by writing `a_decimal_number: f32 = 2`, where `f32` is a 32 bit floating point number. If you had omitted the `f32` and typed `a_decimal_number := 2`, then the type of this variable would actually have been `int`. But `2` is a valid value for both `f32` and `int`, so you can use that `: f32 =` syntax to use `f32`. You can see the `:=` syntax as saying "I wanna create a variable and set it to the right-hand side value, but I don't wanna write the type, please infer it from the right-hand side value".

Moreover, these two lines do the same thing:
```C
a_decimal_number: f32 = 2
a_decimal_number := f32(2)
```

In the second version we are casting the value 2 into an `f32`, which will then be the inferred type of the variable.

The line `for counter < 20 {` makes a loop that runs until the counter reaches the value 20 or more. All `for` and `while` loops you find in C/C++ are possible using the `for` loops in Odin:

- `for {}` is a loop that runs forever, equivalent to `while true {}` in C.
- `for i < 10 {}` is a loop that runs for as long as i is less than 10, equivalent to `while i < 10 {}` in C.
- `for i in 0..<10 {}` is a loop that creates a value i and sets it to 0 and then increases it by 1 until it reaches 10. Equivalent to `for int = 0; i < 10; i += 1 {}` in C.
- `for i := 0; i < 10; i += 1 {}` is a standard for loop. Note that Odin does not support the ++ operator, so you have to write += 1.
- `for item in array_of_items {}` is a "for each" loop that goes over an array and presents you with each item in the array. Similar to `for (auto& item : array_of_items) {}` in C++
- `for item, index in array_of_items {}` same as the previous one, but gives also gives you access to the index of the current item.
- `for &item in array_of_items {}` similar to the previous one, but `item` will be possible to alter, changing that item in the array. 

## Procedure parameters and return values
Procedures can have parameters and return multiple values:

```C
my_proc :: proc(number: int, text: string) -> (int, bool) {
	fmt.printfln("This is the number: %v, this is the text: %v", number, text)
	return len(text), true
}

some_int, some_bool := my_proc(5, "Interesting text")
```

We see here that `my_proc` is a procedure that takes 2 parameters and returns 2 values. The return values can be assigned to new variables.

If you do not want one of the returned values, then you can use `_` to say "I don't care about that one": `some_int, _ := my_proc(5, "More text")`

## If statements

If statements are created like so:

```C
if some_variable == 7 {
	// do stuff
}
```

There's no need for a parenthesis around the condition of the if-statement. As expected you can create `else if` and `else` branches:

```C
if some_variable == 7 {
	// do stuff
} else if some_variable == 10 {
	// do other stuff
} else {
	// do fallback stuff
}
```

If statements without curly braces are _not_ allowed. I.e. this is will not compile:
```C
if some_variable == 7
	some_proc()
```

however, you can do
```C
if some_variable == 7 do some_proc()
```

I tend to just write the full three lines style:
```C
if some_variable == 7 {
	// do stuff
}
```

Some people like putting the `{` on a new line. If you do that then perhaps the `if X do stuff()` syntax is more useful. But I'm just gonna recommend keeping the `{` on the same line, because it plays nicer with Odin's lack of semicolons.

## Constants

You can create compile-time constants by typing

```C
MY_CONSTANT :: 123
```

Constants cannot be changed, as the name implies. They are useful because the compiler can act on their value during compile-time, since it can be certain of their value.

A very nice things about Odin constants is that they are not typed in the same way as variables. For example, you can type:

`my_variable: int = MY_CONSTANT`

or

`my_variable: f32 = MY_CONSTANT`

The compiler has not attached a specific type to this constant, so it is possible to assign it to variables of different types. In this case we say that MY_CONSTANT is an "untyped integer", since it's obviously an integer, but it can in practice fit into several different types.

However if you type

```C
ANOTHER_CONSTANT :: 72.12
```

then `my_variable: int = ANOTHER_CONSTANT` will not compile unless you force the constant into an int, which you can do by writing `my_variable := int(ANOTHER_CONSTANT)`. In doing so the value `72.12` would be truncated into just `72`. Note: Here we moved the `int` to the right hand side because we did a cast, so we didn't have to write the type of the variable.

In this case ANOTHER_CONSTANT is an "untyped floating point number", meaning that it cannot automatically be put into an integer, but it can be put into different kinds of floating point numbers without the need for a cast, like so:

```C
my_f32: f32 = ANOTHER_CONSTANT
my_f64: f64 = ANOTHER_CONSTANT
```

You can think of it like this: Constants can be automatically put into a variable if that variable can accommodate the constant.

> In C constants have stricter types, which is why you have to type `0.5f` all the time to denote a 32 bit floating point constant. Typing `0.5` in C will get you a double precision (64 bit) floating point number. Trying to assign `0.5` to a 32 bit float in C is a compilation error. The constant system in Odin is a bit more comfy. In Odin the compiler will still emit an error if the constant won't fit into the type, such as trying to shove a giant integer into an 8 bit integer type, but you don't have to worry about the specific types of constants as much.

If you really want to, then you can give a constant a type, like so:
```C
A_THIRD_CONSTANT : int : 7000
```

This can be compared to when we explicitly specify the type of a variable:
```C
some_variable: int = 7000
```

In both cases you wedge in the type between the `::` or the `:=`

## Array basics, swizzling and array programming

Say that you want to represent a position in 3D space (I make video games, so this example comes naturally to me), then you can do so using three numbers: 
```C
position: [3]f32
```

This creates a variable `position` that is an array of three `f32` numbers.

You can fetch the different values from `position` like so: `position[0]`. However, when arrays have 4 or fewer items, then you can index them using `x`, `y`, `z` and `w` as well. So instead of `position[0]` you can write `position.x`.

You can also swizzle such arrays: `2d_pos := position.xy` will make `2d_pos` into a variable of type `[2]f32` that contains the x and y parts of the position. You can also do stuff like `position.xxyy` or `position.wyx`. Finally, because in some applications we use 4 dimensional numbers to represent colors, you can also swizzle using `.rgba`.

If you don't wanna write `[3]f32` all the time then you can make an new type instead: 
```C
Vector3 :: [3]f32
```

You can add arrays that have matching item type and size, like so:
```C
Vector3 :: [3]f32
position: Vector3
velocity := Vector3 { 0, 0, 10 }
position += velocity
```

This is known as 'array programming'. You can add arrays, subtract them and multiply them with scalars. Odin does not have operator overloading, but one of the most common use cases for operator overloading is doing vector maths. So I haven't missed operator overloading at all because of how great array programming works.

Things like this is a great example of why I enjoy Odin so much for video games programming.

## Structs

You can group variables by using structs, defining a struct creates a new type that you can later use.

```C
Player :: struct {
	position: Vector3,
	health: int,
}
```

The struct is created by doing `Struct_Name :: struct {}`. Here we see the double colon (`::`) again. So far we have seen three different things use the `::` syntax: procedures, constants and now struct definitions. All these three things are known at compile time, so you can see the `::` as something that creates things that are possible to reason about at compile-time.

The struct has a number of fields, each field looks like a variable declaration, but with a comma at the end.

> In C/C++ I used to have a coding standard to always put a comma even at the last field of a struct. That way, when adding a new field, you didn't forget the comma on the previous line, saving you that compile error. With this in mind, in Odin commas are enforced, even for the last line.

We can create a variable using our struct type like so: 
```C
player: Player
```

The health and the position fields inside this player struct will be all default initialized to zero.

There are no classes in Odin, only structs. Structs can only contain fields, there are no "methods". This is in line with Odin trying to be a better C, and just like in C there are no methods. You write your program using procedures, structs and arrays.

If you want to initialize the player struct when you create it, then you can do like this:

```C
player := Player {
	health = 100,
}
```

This player will then have the health field set to 100. Any field not mentioned will be zero-initialized. In this case the position will be all zeroes.

> These initializers where you mention fields by names are known in C as designated initializers. They are one of my favorite features in C due to it being possible to mention only some fields and the non-mentioned will be zeroed. Unfortunately, in C++ they have only recently been added. So it was always an annoying trade-off when writing C++ code, that you could not use the designated initializers.

We could also have written
```C
player: Player = { blabla
```
But doing `player := Player {` whenever possible is usually considered more idiomatic Odin.

You can replace the value of a whole struct like so:

```C
player := Player {
	health = 100
}

// a bit later

player = {
	health = 7000
}
```

## Pointers and passing proc parameters by pointer/value/reference

Say that you have this code:

```C
my_number := 7

increase_number :: proc(n: ^int, amount: int) {
	n^ += amount
}

increase_number(&my_number, 3)

// my_number is now 10

```

Here we are passing a pointer to `my_number` into the proc `increase_number` so that the proc can alter it. The ^ in front of `^int` means that we want a "pointer to an integer". We get a pointer to something by taking the address of it, using the `&` operator.

Within the proc you can add `amount` to the variable that the pointer points to by putting a ^ on the right side of n, as we see in the `n^ += amount` line. This can be compared to writing `*n += amount` in C.

You can also pass structs by pointer and modify its fields:

```C
Player :: struct {
	position: Vector3,
	health: int,
}

damage_player :: proc(player: ^Player, amount: int) {
	player.health -= amount
}

player := Player {
	health = 100,
}

damage_player(&player, 10)

// player.health is now 90

```

> Here you can see a difference from C/C++: You do not need to use the -> to fetch fields of the pointed-to-struct. You can just use . directly.

Now, say that we want to pass this player struct to a proc, but the proc won't modify the player, so we do not want to pass it by pointer. How would that look? Like this:

```C
Player :: struct {
	position: Vector3,
	health: int,
}

Box :: struct {
	position: Vector3,
	size: Vector3,
}

player_collider :: proc(player: Player) -> Box {
	return {
		position = player.position,
		size = { 0.3, 1.6, 0.3 },
	}
}

player := Player {
	health = 100,
}

collider := player_collider(player)

// use collider for something

```

Here we take the player and return a new struct that tells us how 'big' the player is by combining the position of it and the size of it.

But now those who come from C/C++ might say: Hang on! Doesn't this copy the whole player struct each time you call `player_collider(player)`, since you do not pass it by pointer?

The answer is no, as long as the player struct is larger than pointer size (8 bytes on 64 bit machines), it will automatically be passed as an immutable reference. This means that from within `player_collider` the parameter `player` is the exact same memory as the `player` variable we passed in. But since it is immutable you cannot change it, trying to alter any of the fields of player is a compilation error.

> If you come from C++ you probably do a lot of `const Player& player` in order to pass things by immutable (const) reference. In Odin you simply pass by value/reference (depending on size), or by pointer (if you want the proc to be able to modify the parameter).

Actually, all proc parameters in Odin are _always_ immutable:
```C
wont_compile :: proc(number: int) -> int {
	number += 5
	return number
}
```

This proc won't compile. All proc parameters being immutable makes it possible for the compiler to do optimizations such as the pass-by-reference-automatically.

If you want a local, mutable copy, then you can do:
```C
wont_compile :: proc(number: int) -> int {
	number := number
	number += 5
	return number
}
```
As you see we define a _new variable_ called number, even though the proc parameter number already exists. This is allowed for proc parameters in order to create mutable copies without having to come up with new names for them.


## Dynamic arrays
If you want an array that can grow and shrink while your program runs, then you can use a dynamic array.

Create it like so:
```C
dyn_arr: [dynamic]int
```

Add things to it like so: 
```C
append(&dyn_arr, 5)
```

Remove things like so:
```C
unordered_remove(&dyn_arr, some_index)
// or, if you care about the order of things (slower):
ordered_remove(&dyn_arr, some_index) 
```

The line `dyn_arr: [dynamic]int` will not allocate any memory. However, when you append stuff to an empty dynamic array, then it will need to grow. The program will then use an allocator to allocate memory. More about that in a bit.

What is a dynamic array internally? It's essentially a struct that stores these fields:

```C
data:      rawptr,
len:       int,
cap:       int,
allocator: Allocator,
```

It also remembers what type the items of the dynamic array should be. So when you write `dyn_arr: [dynamic]int` then the `data`, `len`, `cap` and `allocator` will all be default initialized to zero. Later, when `append` is run, it sees that `data` is `0` (or `nil`, which is the zero value for pointers). It will therefore  try to allocate some initial memory for the array. It will check if the `allocator` field is set and use that allocator to allocate the memory, but in the `dyn_arr: [dynamic]int` example, the allocator will be zeroed, so it will instead fall back to using `context.allocator`. It will set `allocator = context.allocator` in this case.

If you want to use some other allocator for your dynamic array, then you can create the dynamic array like this instead:

```C
dyn_arr := make([dynamic]int, my_allocator)
```

This will create a dynamic array this all zeroed expect that the allocator field of it is set to the allocated you passed it.

If you have some proc that creates and returns a dynamic array, then you could make that proc use a custom allocator for the dynamic array like this:

```C
figure_out_stuff :: proc() -> [dynamic]int {
	res: [dynamic]int
	append(&res, 5)
	return res
}

context.allocator = my_allocator
numbers := figure_out_stuff()

// numbers will be allocated using my_allocator
```

More about this `context` stuff soon, but for now we can just note that the `figure_out_stuff()` proc will use the allocator you set on the context.

To free the memory of the dynamic array, write:
```C
delete(dyn_arr)
```

You can also clear it, which just sets the `len` inside the dynamic array to 0:

```C
clear(&dyn_arr)
```

Note that delete takes a value and clear takes a pointer. This is because clear needs to modify a field (it sets `len` to `0`), while delete will only tell the allocator to free memory, but it won't touch any of the fields, it just needs the value of the `data: rawptr` field.

## Slices: A window into a part of an array

Say that a fixed array of integers is created like so:
```C
my_numbers: [128]int
```

Now, if you just want a part of `my_numbers`, say that you only want the first 20 items, then you can fetch those using a slice:

```C
my_numbers: [128]int
first_20 := my_numbers[0:20]
last_20 := my_numbers[len(my_numbers)-20:]
```
A slice is a 'window' that looks at a part of an array. Slices are essentially pointers plus a length, where the pointer says where the slice starts. Creating a slice does not allocate any memory, it uses the same memory as the thing you sliced, but looks into just a part of it. We create a slice using the `[:]` operator, the first index is to the left of the : and the last index is to the right of the `:`

Like you see above, we can skip indices on either side of the colon: `last_20 := my_numbers[len(my_numbers)-20:]`. If you skip the index on the left side then the slice will start at index 0. If you skip the index on the right side then the slice will run until the last index. If you skip both indices then you get a slice that looks at the whole thing. This is useful when you have a proc that expects a slice, but you have a fixed size array or dynamic array.

> In C if you run out of bounds, i.e. using indices that are bigger than the length of the array, your program may continue outside it and start trashing memory. Odin has built in bounds checking of arrays, slices and dynamic arrays. So you get a nice assert immediately when you go out of bounds. This bounds checking does have a slight performance impact, so for for release/production builds you can choose to disable bounds checking using the compiler flag `-no-bounds-check`. I'd say disabling it in release builds is fine, because you'll probably catch most of those out-of-bounds errors while testing your development build anyways.

If you want the slice to have its own memory, then you can import the package `"core:slice"` and run `slice.clone(some_slice)`, which will then be cloned using the allocator `context.allocator`. You can also write `slice.clone(some_slice, another_allocator)` to force the clone to happen with an allocator of your choice.

As an example, say that you want to consider just a part of an array. Say that you only want to display 10 numbers at a time from a bigger list of numbers. You could do this using slices:

```C
display_numbers :: proc(numbers: []int) {
	// code to display the numbers
}

my_numbers: [128]int

for i := 0; i < len(my_numbers); i += 10 {
	display_numbers(my_numbers[i:min(i+10, len(my_numbers))])
}
```

Slices can be formed from different kinds from arrays, other slices and also dynamic arrays.

This means that you can do this:

```C
my_numbers: [128]int
first_20 := my_numbers[0:20]
last_10_of_first_20 := first_20[10:]
```

That is, I created a second slice from my first slice.

You can slice dynamic arrays in the same way:

```C
a_dynamic_array: [dynamic]int

for i in 0..<100 {
	append(&a_dynamic_array, i)
}

first_50 := a_dynamic_array[:50]
```

A good idea is to make all procedures that you want to be reusable take slices, since you can form a slice from all the other types of containers basically for free (remember: making a slice does not allocate memory, it's just a pointer an a length). There's no point in feeding a proc a dynamic array unless you actually want to modify the dynamic array. If you just wanna use the data in it, then feed it a slice, even if it as slice that looks at the whole dynamic array.

## Implicit context and custom allocators
So we've seen the word `context` in the code above. I wrote `context.allocator` a bunch of times. All procedures in Odin get fed the current _context_. It is an extra parameter that is always passed along.

The context is a struct that looks like this (see `odin/base/runtime/core.odin`)
```C
Context :: struct {
	allocator:              Allocator,
	temp_allocator:         Allocator,
	assertion_failure_proc: Assertion_Failure_Proc,
	logger:                 Logger,

	user_ptr:   rawptr,
	user_index: int,

	// Internal use only
	_internal: rawptr,
}
```

The two most used fields of it is `allocator` and `temp_allocator`

> If you don't want a proc to be fed the context you can write `some_proc :: proc "contextless" () {}`. For some super-often run procs it can be a slight optimization. But I would never worry about this unless you are writing low level maths code or similar.

So if you have a proc that allocates memory using `context.allocator` (which is the default when you don't specify an allocator), then you can change the `context.allocator` field before calling it in order to control what allocator it should use. In code:

```C
do_work :: proc() {
	make_lots_of_ints :: proc() -> []int {
		ints := make([]int, 4096)
		ints[5] = 7
		return ints
	}

	context.allocator = my_allocator
	my_ints := make_lots_of_ints()
	// my_ints will be allocated using my_allocator

	// to free this memory, use:
	delete(my_ints)
}
```

`make` will allocate memory using `context.allocator`. If you look at the definition of `make` (see `<your_odin_compiler_dir>/base/runtime/core_builtin.odin`), you'll see that it takes an allocator parameter that has the default value `context.allocator`.

This means that you can also force make to use a specific allocator: `ints := make([]int, 4096, my_allocator)`. Also, what we are doing here is creating a brand new slice of type `[]int` that is 4096 items long. So slices are not always "windows" into other arrays, they can be the original thing that owns its own memory, you can use them as dynamically allocated fixed size arrays.

At the end of the scope the context will be re-set to whatever value it had before the scope started. Note that these scopes are not just the procedure's scopes, but can be any scope, like so:

```
{
	context.allocator = my_allocator
	my_ints := make_lots_of_ints()
}
// context will now be the same as before the scope above started
```

Note that `delete()` is used to free the memory of slices and dynamic arrays alike. For slices it will use `context.allocator` (you can also send in a second parameter that to use a specific allocator). But for dynamic arrays the the dynamic array itself remembers which allocator is used (since it needs to be able to grow when you run `append()`), so for dynamic arrays it does not care about the value of `context.allocator` when running `delete`.

## Parametric polymorphism: Reusable, generic procedures

We've already seen how we can make arrays that contain items of any type. But how can we make code that can operate on _any_ type?

In Odin we can do this using parametric polymorphism. This means that we can make parameters of procedures generic.

Say that we have this procedure that implements clamp (Odin has clamp built in, this is just for the sake of an example):

```C
clamp :: proc(val, min, max: f32) -> f32 {
	if val <= min {
		return min
	}

	if val >= max {
		return max
	}

	return val
}
```

Now, we may want to clamp integers or 64 bit floats as well. We can make our clamp proc generic like this:

```C
clamp :: proc(val, min, max: $T) -> T {
	if val <= min {
		return min
	}

	if val >= max {
		return max
	}

	return val
}
```

The `f32` in the list of parameters has been replaced with a `$T`. This means that the type of `val`, `min` and `max` (they all have to be the same type) will be usable by typing `T`. As you can see we immediately use `T` as the type of the return value of the proc.

This proc can now be used for any numeric type. The compiler will generate the different variations for you as you use clamp with different types of parameters. If it is not possible to generate the code, then you'll get a compilation error, which would happen if you tried to use this proc with a non-numeric type.

Now, say that you want want to create an array of random size (again, silly thing to do perhaps, but it's a simple example):

```C
make_random_sized_slice :: proc($T: typeid, max_size: int) -> []T {
	random_size := rand.int31_max(max_size)
	return make([]T, random_size)
}
```

We could now use this proc like so:

```C
my_slice := make_random_sized_slice(f32, 1000)
```

`my_slice` will then be a slice of type `[]f32` with any size between 0 and 1000.

The difference between these two examples is that in the second example I moved the `$T` to the parameter name instead of having it on the type. We see in the second example that the type of `$T` is typeid, which is the "type of types". We can then use T in the code to refer to this type. Note that we cannot just type `T: typeid` in the parameter list, the `$` has to be there so that the compiler knows we intend to use this value as a compile-time constant. You can have procs that have a parameter `T: typeid`, but then T would not be possible to use at compile-time. We need T to be usable at compile-time because we write `[]T` further down, and `[]T` is itself a type, which must be known at compile-time.

We can also see it like this: In the first example we made a proc that accepts a parameter of a generic type. We can in that proc then use both the value and the type. In the second example we only needed a type, there was no value to send in!

Finally, we can use similar things to create generic structs:

```C
Special_Array :: struct($T: typeid, $N: int) {
	items: [N]T,
	num_items_used: int,
}
```

We see that this generic struct accepts a typeid that is used to choose the type of the `items` array. Similarly, we also put in `$N: int`, which gives us a compile-time value to use for the size of the `items` array.

We can then use this generic struct definition like so:
```C
array: Special_Array(f64, 128)
```
array will then be a variable that is of the `Special_Array` type, where the items array inside it will contain `128` items of type `f64`.

We can make procedures that use these generic structs:
```C
find_random_thing_in_special_array :: proc(arr: Special_Array($T, $N)) -> T {
	return arr.items[rand.int31_max(arr.num_items_used)]
}

array: Special_Array(f64, 128)
random_thing := find_random_thing_in_special_array(array)
```
We see that this proc can figure out the type `T` it needs for the return value from the specialization of the struct we send into it.

You can also limit, or specialize, what types that are allowed, here's [an example from the overview](https://odin-lang.org/docs/overview/#specialization):
```C
make_slice :: proc($T: typeid/[]$E, len: int) -> T {
	return make(T, len)
}
```
This proc makes slices of a certain type. But the `/` just after the typeid limits the possible types it accepts. In this case `T` must be a `typeid` that is a slice, but the item type of the slice is still generic.

## Getting comfy with manual memory management

We've already seen how you can switch out `context.allocator` and how you can `delete` memory you've allocated previously.

So in a sense, whenever you need dynamic memory, be it for a dynamic array or for creating a new slice, then just use `make([dynamic]int)` or `make([]int, 1024)`, which will use `context.allocator`.

When you don't want that memory anymore, then you call `delete(your_dynamically_allocated_thing)`

### Tracking memory leaks

However, a common problem is that you might forget `delete`, which may lead to a memory leak. For something that is allocated once at startup, a memory leak is of little concern. But if you allocate memory over and over and always forget to delete it, then your program's memory usage will continuously grow.

To find memory leaks, I would advice that you use a tracking allocator. Here's how to do it (this example is from the [overview](https://odin-lang.org/docs/overview/#when-statements)):
```C
package main

import "core:fmt"
import "core:mem"

main :: proc() {
	when ODIN_DEBUG {
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
			if len(track.bad_free_array) > 0 {
				fmt.eprintf("=== %v incorrect frees: ===\n", len(track.bad_free_array))
				for entry in track.bad_free_array {
					fmt.eprintf("- %p @ %v\n", entry.memory, entry.location)
				}
			}
			mem.tracking_allocator_destroy(&track)
		}
	}
	
	
	do_stuff()
}
```

What this code does is swap out `context.allocator` for a a tracking allocator. The tracking allocator "wraps" the original `context.allocator`, but it saves all allocations on a big list along with information of on what line the allocation happened etc.

When your program shuts down, it will check if anything is still inside that big list, and if it is then it will print a warning about this. This makes manual memory management much less scary. It will also write warning if you did any 'bad frees', which means trying to free memory that wasn't actually allocated.

A bit about how the `defer` and `when` stuff above works:

The `defer { /* the tracking allocator setup */ }` line tells the program to run the code between those curly braces when the current scope ends, which will happen at the end of the `main` proc. Note that the tracking allocator setup happens inside a `when ODIN_DEBUG {` block. `when` is used to include and exclude code based on values of constants. `ODIN_DEBUG` is a constant that is only true when the `-debug` compiler flag is set, which is a good idea for 'development' builds (it also makes the program generate debug info, so that you can use a debugger such as VSCode, RemedyBG or RAD Debugger).

This means that the setup and checking of the tracking allocator will only happen if you compile with `-debug`.

Note that the scopes associated with `when` do not work like normal scopes. The `defer` statement will wait until the end of the `main` proc before it runs the 'deferred' code, it will not run it at the end of the scope associated with the `when`.

This does not mean that `defer` waits until the end of the proc, it will wait until the end of the current scope. So if the program looked like this:
```C
main :: proc() {
	// note: anonymous scope starts here
	{
		when ODIN_DEBUG {
			number: int
			defer {
				fmt.println("hello")
			}
		}

		// you can still use number here:
		number = 7

	} // hello will print at this curly brace
	
	do_stuff()
}
```

This example shows that the scope associated with `when` does not work like a normal scope, it is only there to group code that should be included when the condition is true, but it does not create a real scope. Therefore I tend to avoid calling them scopes, instead perhaps we should call them "when blocks".

You can also watch this video I made on the tracking allocator:
{{<youtube dg6qogN8kIE>}}

### Temp allocator

Many times you need to allocate some dynamic memory, but you do not need it later. Some algorithms may require a dynamic array to do some processing, but the dynamic array is not needed when the algorithm finishes.

So if you have code like this:

```C
great_algorithm :: proc() -> int {
	numbers: [dynamic]int

	for i in 0..<100 {
		append(&numbers, i)
	}

	rand.shuffle(numbers[:])
	n := numbers[0]
	delete(numbers)
	return n
}
```

It's a silly example, but it's simple to talk about. `append()` will need to grow the dynamic array. As you see we shuffle (randomize) the array and then pick the first thing in it and then return it. Before we return we delete the dynamic array, avoiding a memory leak.

In this kind of situation, I would instead use the temp allocator:

```C
great_algorithm :: proc() -> int {
	numbers: make([dynamic]int, context.temp_allocator)

	for i in 0..<100 {
		append(&numbers, i)
	}

	slice.shuffle(numbers[:])
	return numbers[0]
}
```

I.e. we tell the dynamic array to use the `context.temp_allocator` to allocate memory. This means that we do not 'care' about this memory in the long run.

For how long is the memory allocated using the temp allocator valid? Until you run `free_all(context.temp_allocator)`. It is up to you to put this line at some place in your program. It will free all the memory in the temp allocator. I make video games and such programs usually have a 'main loop', so for video games it is convenient to put the `free_all` as the last line of the main loop. This means that you can expect all temp allocated memory to be valid until the end of the current 'main loop lap' (known as the current frame).

Do not forget the `free_all` line, if you do and you use the temp allocator, then you will get memory leaks and see every increasing memory usage.

> TECHNICALITY: I said that `free_all(context.temp_allocator)` frees all the memory on the temp allocator, but for the default temp allocator this is half-true. That allocator uses blocks to put your temp allocations in. If the current block is full, then it makes a new block. When `free_all` is run, then it frees all the blocks except for the first one, since it can reuse that. This is just an optimization, since memory allocations can be slow it is a good idea to just reuse the first block of the temp allocator.

The combination tracking long-term allocations and using temp allocator for short term things makes manual memory management very comfy.

## Strings

Strings in Odin use UTF8. This is different from using ASCII, where characters are always a byte. In UTF8 a character can be of varying memory size. English characters still just use single byte, but characters from other languages can use be bigger in memory. If you want a quick overview of this, then you can read [this entertaining article](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/).

You can create and loop over a string:

```C
my_string := "Hi, hello. How are you?"

for r in my_string {
	// here r is of type rune
}
```

This loop will loop over the string UTF8 character by character, not byte-by-byte. `r` is of type `rune`, this is Odin's type for denoting a single UTF8 character. In the package `core:unicode/utf8` you will find lots of nice procedures to work with runes.

## Package system and code organization

When you compile and run your program using `odin run .` then it takes all files in the current directory (`.`) and compiles them as a single package.

When you do
```C
import "core:fmt"
```
then you are importing the package `fmt` from the standard library. This means that all the stuff in the folder `<your_odin_compiler_dir>/core/fmt` will be available under the name `fmt`.

If you want to have some different name for an import, then you can rename them when importing. This is common when importing the raylib bindings (a library for creating videos games):
```C
import rl "vendor:raylib"
```
Now all those things from the folder `<your_odin_compiler_dir>/vendor/raylib` are available under the name `rl`

Because of this unfixable name conflicts between packages can never happen. If two packages use the same folder name like so:
```
import "core:fmt"
import "my_localization_system/fmt"
```
then you can fix this by just giving one of them a local alias:
```
import "core:fmt"
import lfmt "my_localization_system/fmt"
```

In Odin the program that uses packages can never end up in a state where the packages themselves pollute a global namespace with names that collide, since everything inside them end up under these prefixes such as `fmt.` and `rl.`

> Those who have programmed C a lot can testify that name collisions between libraries do happen. Because of this C libs often have prefixes on all the stuff they expose. In Odin the prefix is instead chosen at the import site, which is more robust.

You can create local packages within your project by just creating a folder and importing it:
```
import xp "xml_parser"
```

### Use packages for libraries, not as namespaces
I would encourage you to only split things out to separate packages if those  things are absolutely independent of your project. So for example a generic XML parser could be such a package. But if you just want to compartmentalize some code then I recommend to keep that code within your main program package and instead add prefixes to proc names etc, just like in C.

The Odin package system does not allow cyclic dependencies, so package `A` can import package `B` only if package `B` does not import package `A`. With this in mind, I would say that the Odin package system is mainly for creating _libraries_ that are totally independent.

That said, there are some people who have successfully used packages to compartmentalize their project. But in my experience it just leads to pain and suffering because you will suddenly need a cyclic dependency, and you can't have it.

Some may balk at the idea of using prefixes within a project, they want to use packages as some sort of namespace but still have cyclic dependencies. I would encourage you to just try using prefixed proc names etc, it is not that big of a deal. This is in line with Odin trying to be a better C while still keeping things very simple.

## Where to find more Odin resources

There's a great list of resources, libraries and software on [Jakub Tomšů's Awesome Odin page](https://github.com/jakubtomsu/awesome-odin) 

If you like video content, then I made this 90 minute video where I create a little video game from start to finish:

{{<youtube lfiQNCNUifI>}}

## Thanks for reading!

If you liked this article and want to support me, then you can [become a patron](https://www.patreon.com/karl_zylinski).

If you have any questions or wanna discuss what I wrote, then you can [join my Discord server](https://discord.gg/4FsHgtBmFK).

Have a nice day!
/Karl Zylinski
