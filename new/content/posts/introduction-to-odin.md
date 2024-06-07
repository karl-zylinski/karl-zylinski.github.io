---
title: "Introduction to the Odin Programming Language"
date: 2024-06-07T00:01:00+02:00

draft: false

#cover:
#  image: "/odinraylib2/cover.png"
---

## Introduction

This is a book that aims to be an introduction the Odin Programming Language. It is aimed at people who know a bit of programming, but have never touched Odin. There will be some notes on differences to C/C++, as Odin in many ways tries to be better C.

Over the last few years almost all my hobby programming and more recently, my professional programming, has been done using the Odin Programming Language, sometimes referred to as Odinlang. Since a year back I create my own video games using Odinlang, you can find my game [CAT & ONION](https://store.steampowered.com/app/2781210/CAT__ONION/) on Steam, it is completely created in Odin.

Why Odin? I come from having worked on video game engines where I programmed a lot in C++ and C. While those languages are very powerful, they are also very old and have a lot of baggage. Odin strives to be a modern alternative to C, it doesn't try to fundamentally solve any new problems. It is not a Big Agenda language, meaning that it doesn't try to enforce a new way of programming onto you.

<!--> My C++ programming was done mostly in a C-like way, I used C++ to get a "C with some sugar on top". This "back-to-C" style of programming C++ is quite popular among game developers.-->

When I first started programming Odin it felt like it point-for-point fixed many of the issues I had with C. Instead of listing those issues here, I will point discuss these differences with special "if you come from C" notes throughout this book.
<!--
- Zero initialization. Odin initializes all memory to zero by default unless you tell it to not do so. The language and the core library (Odin's standard library) also tries to use the zero value as a 'good default' as much as possible, which is known as zero-is-initialized. Zero-is-initialized and automatic zero initialization is a comfy combo.
- Multiple return values. In C I often had to create new struct types just to do multiple returns, or use "out parameters". Odin supports multiple return values where the return values can be distributed into several variable on the calling side.
- Generics and parametric polymorphism. In C you have to rely on macros to create generic behavior. In C++ you can use templates. Odin lets you 

Some of these may seem insignificant, but the smart thing about Odin design lies in how carefully the design decisions have been done, it's all very consistent.-->

## A note on learning Odin

While Odin is a great language, the area it is perhaps most lacking in currently is documentation. There is lots of documentation, but it is scattered and incomplete. This book tries to address that by giving a holistic introduction. This book aside, here is my recommendations of where to look if you are stuck and trying to figure out how to do something specific in Odin, I recommend you check these resources in the order stated:

1. Check the [overview](https://odin-lang.org/docs/overview/). The overview is not complete, but searching around in there will answer many beginner questions.
2. Look inside [demo.odin](https://github.com/odin-lang/Odin/blob/master/examples/demo/demo.odin). It is an example that comes with the compiler that tries to showcase many different features of the language.
3. Search in the core library. I.e. pull the core folder inside your Odin install into your text editor and just search around in there. The core library is well written and easy to read, it is a great way to learn many Odin concepts.
4. Ask in the #beginners channel on the Odin Discord server. The server has thousands of members and many questions get answered very quickly.

## Installing Odin

I have a video on how to setup Odin from source, but I will also go over how to do it below.

{{<youtube yq5VabsGz_4>}}

There are two ways to install Odin. First way is to do it from a binary release and the second way is to build the compiler from source.

BLABLA

Go to the [Getting Started](https://odin-lang.org/docs/install/) page on the Odin website. On there you'll find both binary versions of Odin as well as how to set the compiler up from source.

## Syntax

### Basic "Hello World"

Here is a tiny Hello World program written in Odin:
```C
package hello_world

import "core:fmt"

main :: proc() {
	fmt.println("Hello World!")
} 
```

Functions are in Odin referred to as procedures. `main` is the default entry point for the program. We see that procedures are created by typing `procedure_name :: proc() {}`

> The reason for saying procedures, or "procs", instead of functions is that the word function, if one wants to be nit-picky, is a bunch of code that has _no side effects_. This is why when we talk about "functional programming" we mean code that composes new functionality from functions, where those functions do not modify and global state. Now, functions can be seen as a special case of procedures. I.e. a procedure can take some parameters, return some values and also have side-effects on global variables. Functions can be seen as a sub-species of procedures.

The line `import "core:fmt"` fetches the `fmt` package from the standard library, which contains procedures that can print to the console and also format strings. All those things inside the `fmt` package can then be used by typing `fmt.some_procedure_name()`, which we see inside the `main` proc where we use the `println` proc to print to the console.

Semicolons are not needed, and I would encourage to simply not use them. Not using semicolons fits well with how Odin looks. You can still use them if you really want to! There is also a compiler flag to make it an error if you put in unnecessary semicolons (`-vet-semicolon`)

If you copy-paste the code above into a file and save that file inside a new folder, then you can from the command prompt navigate to that folder and run your program by typing:
`odin run .`

This will take all the files within the current directory, i.e. `.` refers to the current directory, and compile them as a single package, output an executable and run it.

I will say more about the package system later, but for now I just want to point out that the line `package hello_world` must be the same for all odin files within a folder.

### Variables and loops

```C
package hello_world

import "core:fmt"

main :: proc() {
	a_string := "Hello World!"
	fmt.println(a_string)

	counter: int

	for counter < 20 {
		fmt.fprint("Counter value: %v", counter)
		counter += 1
	}
} 
```

The line `a_string := "Hello World!"` declares and initializes a variable. Odin is a strongly typed language, meaning that all variables must have a specific type, but the type can be inferred from the right-hand side, which happens here. In this case `a_string` will be of type `string`.

The line `counter: int` also declares a variable, but since no value is provided for it, it will be initialized to zero. If you for some reason do not want to have it initialized, then you can write `counter: int ---`. However, in almost all cases you will want zero initialization, in some performance sensitive algorithms you may want to skip it, but I have almost never had to do so.

> Zero initialization. Odin initializes all memory to zero by default unless you tell it to not do so. The language and the core library (Odin's standard library) also tries to use the zero value as a 'good default' as much as possible, which is known as zero-is-initialized. Zero-is-initialized and automatic zero initialization is a comfy combo.

> A_BETTER_C: In C and C++ if you write `int counter;` then the variable counter will not be initialized and can just contain garbage memory, which is almost never what you want.

We can look closer that the how the usage of `:` and `=` varies when declaring and setting variable values:

```C
a_string := "Hello World!"
counter: int
a_decimal_number: f32 = 2
a_decimal_number = 3.2
```

Only having a `=`, like the last line, is only valid if there is an already existing variable with that name. You can also force a variable to have specific type by writing `a_decimal_number: f32 = 2`, where `f32` is a 32 bit floating point number. If you had omitted the f32 and typed `a_decimal_number := 2`, then the type of this variable would actually have been `int`. But `2` is a valid value for both `f32` and `int`, so you can use that `: f32 =` syntax to force the type. So you can see the `:=` syntax as saying "I don't wanna write the type, please infer it from the right hand side stuff".

Moreover, these two lines do the same thing:
```C
a_decimal_number: f32 = 2
```

```
a_decimal_number := f32(2)
```

In the second version we are casting the value 2 into an `f32`, which will then be the inferred type of the variable.

The line `for counter < 20 {` makes a loop that runs until counter reaches the value 20 or more. All those `for` and `while` you'll find in C++ are possible using the `for` loops in Odin:

- `for {}` is a loop that runs forever, equivalent to `while true {}` in C.
- `for i < 10 {}` is a loop that runs for as long as i is less than 10, equivalent to `while i < 10 {}` in C.
- `for i in 0..<10 {}` is a loop that creates a value i and sets it to 0 and then increases it by 1 until it reaches 10. Equivalent to `for int = 0; i < 10; i += 1 {}` in C.
- `for item in array_of_items {}` is a "for each" loop that goes over an array and presents you with each item in the array. Similar to `for (auto& item : array_of_items) {}` in C++
- `for item, index in array_of_items {}` same as the previous one, but gives also gives you access to the index of the current item.
- `for &item in array_of_items {}` similar to the previous one, but `item` will be possible to alter, changing that item in the array. 

### If statements

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

### Constants

You can create compile-time constants by typing

```C
MY_CONSTANT :: 123
```

Constants can as the name hints, never be changed. They are useful because the compiler can act on their value during compile-time, since it can be certain of their value.

A very nice things about Odin constants is that they are not typed in the same way as variables.

For example, you can type

`my_variable: int = MY_CONSTANT`

or

`my_variable: f32 = MY_CONSTANT`

The compiler has not attached a specific type to this constant until you try to assign it, so it is possible to assign it several different types. In this case we say that MY_CONSTANT is an "untyped integer", since it's obviously an integer, but it can in practice fit into several different types.

However if you type

```C
ANOTHER_CONSTANT :: 72.12
```

then
`my_variable: int = ANOTHER_CONSTANT`

will not compile unless you force the constant into an int, which you can do by writing
`my_variable := int(ANOTHER_CONSTANT)`.

Note: Here we moved the `int` to the right hand side because we did a cast, so we didn't have to write the type.

In this case ANOTHER_CONSTANT is an "untyped floating point number", meaning that it cannot automatically be put into an integer, but it can be put into different kinds of floating point numbers, like so:

```C
my_f32: f32 = ANOTHER_CONSTANT
my_f64: f64 = ANOTHER_CONSTANT
```

You can think of it as constants being possible to automatically put into any variable if that variable can accommodate the constant.

> A_BETTER_C: In C constants have stricter types, which is why you have to type `0.5f` all the time to denote a 32 bit floating point constant. Typing `0.5` in C will get you a double precision (64 bit) floating point number. The constant system in Odin is a bit more comfy. In Odin the compiler will still emit an error if the constant won't fit into the type, such as trying to shove a giant integer into an 8 bit integer type, but you don't have to worry about the specific types of constants as much.

If you really want to, then you can give a constant a type, like so:
```C
A_THIRD_CONSTANT : int : 7000
```

This can be compared to when we explicitly specify the type of a variable:
```C
some_variable: int = 7000
```

In both cases you wedge in the type between the `::` or the `:=`

### Structs

You can group variables by using structs, defining a struct creates a new type that you can later use.

```C
Player :: struct {
	position: Vector3,
	health: int,
}
```

The struct is created by doing `Struct_Name :: struct {}`. Here we see the double colon again. So far we have seen three different things use the `::` syntax: procedures, constants and now struct definitions. All these three things are known at compile time, so you can see the `::` as something that creates things that are possible to reason about at compile-time.

The struct has a number of fields, each field looks like a variable declaration, but with a comma at the end.

> A_BETTER_C: In C I used to have a coding standard to always put a comma even at the last field of a struct. That way, when you adding in a new field, you didn't forget the comma on the previous line, saving you that compile error. In Odin commas are enforced, even for the last line.

We can create a variable using our struct type like so:

`player: Player`

The health and the position fields inside this player struct will be all default initialized to zero.

There are no classes in Odin, only structs. Structs can only contain fields, there are no "methods". This is in line with Odin trying to be a better C, and just like in C there are no methods.

You can create and initialize the player variable by instead doing this:

```C
player := Player {
	health = 100,
}
```

This player will then have the health field set to 100. Any field not mentioned will be zero-initialized.

> A_BETTER_C: These initializers where you mention fields by names are known in C as designated initializers. They are one of my favorite features in C. Unfortunately, in C++ they have only recently been added. So it was always an annoying trade-off when writing C++ code, that you could not use the designated initializers.

We could also have written
`player: Player = { blabla`

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

### Array basics, swizzling and array programming

Say that you want to represent a position in 3D space, then you can do so using three numbers:

`position: [3]f32`

This creates a variable `position` that is an array of three `f32` numbers.

You can fetch the different values from `position` like so: `position[0]`. However, when arrays have size 4 or smaller, then you can index then using `x`, `y`, `z` and `w` as well:

so `position[0]` is equivalent to `position.x`

furthermore you can swizzle this position: `2d_pos := position.xy` will make `2d_pos` into a variable of type `[2]f32` that contains the x and y parts of the position. You can also do stuff like `position.xxyy` or `position.wyx`. Finally, because in some applications we use 4 dimensional numbers to represent colors, you can also swizzle using `.rgba`.

If you don't wanna write `[3]f32` all the time then you can make an new type instead: `Vector3 :: [3]f32`  

You can add arrays that have matching item type and size, like so:

`position += velocity`

This is known as 'array programming'. You can add arrays, subtract them and multiply them with scalars. Odin does not have operator overloading, but one of the most common use cases for operator overloading is doing vector maths, so I haven't missed operator overloading at all because of how great array programming works.

### More about arrays: Arrays, slices, dynamic arrays


If you want more syntax examples, then I recommend looking at the demo.odin file: https://github.com/odin-lang/Odin/blob/master/examples/demo/demo.odin



<!-- 

Imports can be renamed within the file by typing `import f "core:fmt"`, which makes it possible to write `f.println` instead of `fmt.println`.
Odin has a p

The `.` here needs some explaining, which leads me to say a few words about the package system.

`odin run .` compiles all the files within the current directory as a single package, outputs it all as an executable and runs it. You could also type `odin build .`, which will create an executable that you can later run manually.

Everything within a folder is considered to be part of the same package, the line at the top of the program that says `package hello_world` must also be the same for all the files within that package folder.


-->