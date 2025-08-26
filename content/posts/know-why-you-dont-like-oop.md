---
title: "Know why you don't like OOP"
date: 2025-08-26T22:15:00+02:00

#cover:
#  image: "/temporary-allocator/cover.png"

draft: true
---

Programmers tend to fight about why Object-Oriented Programming (OOP) is good or bad.

Among the anti-OOP crowd, I often see people hate on OOP and "rebroadcast" what they've heard experienced programmers say. But when challenged to explain why OOP is bad, they have a hard time explaining it.

That's probably because they are fans of those experienced programmers, and just want to fit in.

I want to take this in a postitive direction and say: Just write your code in a way that makes sense to you. Avoid things that you know, from experience, to be bad.

# something about it being less black and white, less ANTI OOP vs OOP

So let me share my experiences. In the following sections I'll go over some OOP concepts and why I think they are either "fine" or "bad".

## Interfaces: Fine
Interfaces are used in many big code-bases, OOP or not. If you have a program that wants to support multiple rendering APIs (Direct3D, Vulkan, OpenGL), then that that's a great use of interfaces. Providing support for allocators is another great example. You don't need any OOP features in the language to implement an interface. It's just a struct with some function pointers inside. 

## Methods: Fine
I have no concrete experience that methods are inherently bad. People like to argue about methods a lot. It's one of those discussions that dumbfound me a bit.

## Encapsulation: It depends
Hiding things for the sake of hiding them will just annoy powerusers in the end. I can see the value of sometimes having a black-box API. Some libraries are just some code you drop into a project and use, it may not even be precompiled, perhaps it's just a source file. Then you don't need any encapsulation. But sometimes you want to create a strict, versioned, black-box API where you want to avoid breaking changes. In that case hiding internal things can be good, so you know exactly what parts the end-users are exposed to. That's the API surface, that's the stuff you must be careful with changing.

## Inheritance: Bad
Here I have hard evidence.

Say that you have an array that looks like this in C++ `Array<Entity*>`. The array elements are of pointer type.

> In more high-level languages such as C#, you don't say that it is a pointer explicitly. It kinda always is, as long as it is a class. That's even worse. Those languages are inherently broken.

The reason, from an OOP perspective, may be that Entity is a base class. You can't just do `Array<Entity>`, because the classes that inherit from Entity will vary in size. In order to add to the array you probably do something like:
```C
// entities is of type Array<Entity*>
entities.add(new Entity(blabla));
```

This means that your entities end up all over the place in memory, since each one is separately allocated. That's _terrible_ for performance. CPUs have cached inside them. While iterating over an array, those caches can only be filled properly if things are laid out in a predictable way. Going to RAM instead of the CPU cache can be orders of magnitude slower.

So that's hard evidence that inheritance leads to separate allocations, which leads to terrible performance. So I suggest you stop doing that and just have `Array<Entity>` instead. But then you _cannot_ use intertance. If you are in an OOP-language, then by all means use methods and interfaces. They are great! But avoid separately allocating objects in big arrays.

> If you have an array with a small number of items that you don't iterate often, then you can still do something like `Array<A_Type*>`. That's also a good thing to think about: Understand when you have a big collection of many things that you want to iterate. In those cases, don't use interitance.

## Model things after real world?! Who even does this?

You went to school and leart that Animal is a base class and Cat is a subclass. Sure. But ridiculing OOP people over this is quite silly. Most OOP code-bases use classes that describe the data that the program needs. Pushing that OOP is bad because "modelling after real world is silly" will just make OOP people ignore your furher arguments, since you are just ridiculing them over a strange corner case.

## Conclusion

I say: Use methods and inheritance, it's fine. Avoid inheritance. But also understand yourself why these things may be good/bad. Make up your own mind. Don't just rebroadcast what some YouTuber (including myself) told you.

If you want to learn low-level programming and the Odin programming language, then check out my book "Understanding the Odin Programming Language": https://odinbook.com/

Have a nice day!
/Karl Zylinski