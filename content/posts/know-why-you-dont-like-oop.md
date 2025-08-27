---
title: "Know why you don't like OOP"
date: 2025-08-26T22:15:00+02:00

cover:
  image: "/oop/cover.png"
---

Programmers tend to fight about why Object-Oriented Programming (OOP) is good or bad.

Among the anti-OOP crowd, I often see junior programmers hate on OOP and "rebroadcast" what they've heard experienced programmers say. But when challenged to explain why OOP is bad, they have a hard time explaining it. It's usually because they haven't really experienced those things first hand.

I'd like to take a positive spin on this and say: Just write your code in a way that makes sense to you. Just make sure to avoid things that you _know_ are bad.

With that in mind, I'd like to share my experiences. I won't say "don't use any of the OOP things". Instead I'll state which parts of OOP I think are fine, and which parts I think are bad. If they are bad I will state my reasons. You can then investigate those reasons yourself and make up your own mind. I'll go over these five things:

- Interfaces
- Methods
- Encapsulation
- Inheritance
- Modelling after the real world

## Interfaces: Fine
Interfaces are used in many big code-bases, OOP or not. If you have a program that wants to support multiple rendering APIs (Direct3D, Vulkan, OpenGL), then that that's a great use of interfaces.

Providing support for allocators is another great example. In that case you want some kind of generic interface with multiple implementations. One implementation for each kind of allocator. You can then feed such interface implementations into functions and easily switch allocator.

You don't need any OOP features in the language to implement an interface. In C you can create an interface with with a simple struct that contains some function pointers.

> My readers who use the Odin Programming Language can note that the `Allocator` type in Odin is an interface.

## Methods: Fine
I have no concrete experience that methods are inherently bad. People like to argue about methods a lot. It's one of those discussions that dumbfound me a bit. It's just another way of organizing your code. If you like it and your language supports it, then just use it. I do not see methods as the big problem in OOP.

## Encapsulation: It depends
Hiding things for the sake of hiding them will just create friction. For example, when you know that a private field contains useful state, but you can't access it, then it's just annoying, with little benefit.

However, I can see the value of sometimes having a black-box API. Perhaps you want a strict, versioned API. You want to avoid breaking changes in the API. In that case, hiding internal things can be good. That way you know exactly what parts the end-users are exposed to. That's the API surface. That's the stuff you must be careful with changing.

I'd default to everything being public and never using encapsulation. It's just annoying. But you probably know when you need to create a black-box API. And in that case you'll end up with encapsulation as a byproduct.

## Inheritance: Often bad
I think inheritance in bad for most high-performance use cases. Let's talk about why.

Say that you have an array that looks like this in C++: `Array<Entity*>`. The array elements are of pointer type.

The reason for using a pointer type, from an OOP perspective, may be that `Entity` is a base class. You can't just do `Array<Entity>`, because the subclasses of `Entity` will vary in size. So in order to add to the array, you probably do something like:
```C
// entities is of type Array<Entity*>
entities.add(new Some_Entity_Subclass(some, constructor, parameters));
```

This means that your array items will end up all over the place in memory, since each one is separately allocated. That can be _terrible_ for performance. Why? CPUs have things called _caches_ inside them. While iterating over an array, those caches can only be filled properly if things are laid out in a predictable way. Lots of separately allocated things is the opposite of a predictable memory pattern. So the CPU cache can't be filled properly. Having the computer go to RAM instead of the CPU cache is orders of magnitude slower.

So: Inheritance leads to separate allocations, which leads to bad performance. So I suggest to just do `Array<Entity>`. No elements of pointer type! This will lead to all the items in the array being laid out one-next-to-the-other in memory. Very predictable. Your CPU will love you. But then you _cannot_ use inheritance. If you are in an OOP-language, then by all means use methods and interfaces. But avoid separately allocating lots of objects because you want to use inheritance.

If you have an array with a small number of items, then you can still do something like `Array<Some_Type*>` and separately allocate them. It's not going to matter. Understand when you have high-performance requirements, and when you do not. _If you iterate a huge array very often, then you probably don't want it to be slow_.

In my book [Understanding the Odin Programming Language](https://odinbook.com/) I have a whole chapter on _data-oriented design_. In that chapter I go deeper into these issues and provide benchmarks.

## Modelling after the real world: Who even does this?

You went to school and learnt in the first OOP lecture that `Animal` is a base class and `Cat` is a sub class. Sure. But ridiculing OOP over this is quite silly. Most OOP code-bases use classes in order to describe the data that the program needs, just like any other code base would. Pushing that OOP is bad because "OOP people try to model everything after the real world" will just make OOP people ignore you, since you are just ridiculing them over a strange corner case.

## Conclusion

My opinion is this: Use methods and interfaces if you want, it's fine. Avoid inheritance, especially for large arrays that you iterate often. But also understand _why_. Make up your own mind, based on actual experience. Don't just rebroadcast what some YouTuber (including myself) told you.

Thanks for reading, have a nice day!
/Karl Zylinski