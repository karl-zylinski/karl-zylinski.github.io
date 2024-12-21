---
title: "Data-oriented ideas for small gamedev teams"
date: 2024-08-15T20:45:45+02:00

cover:
  image: "/game_go_brr.png"
---

If you are a game developer then you might have heard about data-oriented design. You might have read that programming in a data-oriented way can make your game run faster.

However, if you come from some object oriented approach then the ideas you've seen can feel a bit overwhelming. It may seem like you need to completely redo all your gameplay code. Where do you even start?

So let's try to keep it simple. Let's look at a single principle that is a good starting point for your data-oriented journey. If you are a small gamedev team, then it may well turn out that it is the only data-oriented principle you need to implement.

> Note: These ideas are in general easier to implement in languages that do manual memory management such as C++, C, Zig or Odin.

After the principle and some discussion about problems that may occur, I just have some short closing words on how far it is reasonable to take these things. Let's go!

## The principle: Avoid separately allocating objects that live in an array

If you have some C++ code (or similar language) where you store your entities in an array like this `Array<Entity*>`, then instead use `Array<Entity>`. I.e. remove that `*` so we store Entity objects instead of pointers to Entity objects.

What's the practical difference? When adding objects to this array, then in the first case you probably do something like `entities.add(new Entity(parameters))`. In the second case you do something like `entities.add(Entity(parameters))`. The difference between the two is that the first uses `new` while the second one doesn't.

In the first case every entity is separately heap allocated and ends up just about anywhere in the computer's memory. In the second case all the entities live within the array itself, meaning that all the entities live one-next-to-the-other within the array.

> NOTE: Doing `new` doesn't _really have to mean that heap allocations occur_. In some languages you can use custom allocators in and in that case you could have `new` allocate into some kind of memory arena. But the default behavior in most language would be a heap allocation.

Using something like `Array<Entity>`, i.e. not heap allocating the separate elements, _is much faster_. Why? Because of a thing called the CPU cache. Imagine that your code loops over the `Array<Entity>`. When the computer fetches the memory related to one entity, then that data ends up in the CPU cache, which is a tiny piece of memory that is physically very close to the CPU and also very fast. However, since the entities are laid out in such a simple way (next to one another in the array) it is also able to _prefetch_ additional memory from the array and also put that into the cache. So when it gets to the next lap of the loop, then it probably already has data related to the next entity in the cache, so it just uses it straight away. Using the cache right away is a lot faster than having to go to the "normal" memory and fetch another item.

This is also why iterating something like `Array<Entity*>` is so much slower: The separately heap allocated entities can be anywhere in memory, they are not laid out in a simple way in the computer's memory. So when memory for one entity is fetched, then it has no good way to predict where data related to the next entity is. When it gets to the next lap of the loop, then the cache probably doesn't contain anything related to the next entity. This means that the cache has to be refilled on almost every lap of the loop. Such a refill is known as a "cache miss". It means that your CPU sits and waits for the data to be fetched from memory.

This example talks about some `Entity` type. But really, this advice is universal: Avoid separately allocating objects that live in an array. Pack them tightly. Your CPU loves it! I will continue to use "entity" in the discussion below, but most of these ideas are universal for any type.

This advice is all good and simple. But the object oriented programmer will run into some issues. Let's look at two of them.

## Problem 1: You can't use inheritance

Something like `Array<Entity>` assumes all object of type Entity are of equal memory size. On the other hand, when you use inheritance you'll have objects of different sizes, which is why you need to do `Array<Entity*>`. Why? Because a subtype of `Entity` may have more fields, so it is bigger. So in order for inheritance to really work, then you need to separately allocate each entity and just store the pointer to that entity in the array. But like I said, we don't want that, since that scatters the elements of the array in memory, causing cache misses.

**Solution: Do not use inheritance.** Instead you can use unions and enums within your Entity struct to store data for different types of entities. There are lots of different ways to do this. Find one that works nicely for your game in particular. I like putting things that many entities have in common straight into the entity struct, and then I use a union for things that only some types of entities use.

## Problem 2: Pointers to elements get invalidated

When you use `Array<Entity*>` then you can (almost) safely store a pointer to an element since they are all separately allocated. But when you use `Array<Entity>` then any pointer to an element inside the array points to data within the array itself. So if the array needs to grow (which it does every now and then when you add things to it), then it may move all of the array's memory to a new location. Then your pointer suddenly points to invalid memory. It's gonna crash!

**Solution: If you need to store a permanent reference to an array element, then use handles, not pointers.** A handle is a thing that contains the index of an array element plus a "generation". The generation tells us if the handle is still valid. If the generation on the handle matches the generation of the item in the array, then it is still valid. If you need to permanently store a reference to an entity, then you store a handle to it, never a pointer. Whenever you need to use the entity that a handle refers to, then you fetch the entity using the handle. Here's a classic article that details the thinking around this: https://floooh.github.io/2018/06/17/handles-vs-pointers.html

There's a thing to realize here: Even if you use `Array<Entity*>` where all objects are separately allocated, then you can still benefit from using handles. This is because some part of your program can destroy an entity while some other part of it has a pointer to it. Then you have a dangling pointer. Using handles prevents that, since the generation within the handle keeps track of if the handle is still valid.

> Note: If you don't have any need for storing permanent references, then you do not need handles.

I have an example implementation of a handle-based array here: https://gist.github.com/karl-zylinski/a5c6acd551473f90b872f46a2fa58deb In the comments at the top of the file there are also some links to some other implementations.

## Knowing when to stop

Once you start going down the route of trying to write more cache-friendly code it is very easy to constantly try to take it one step further. Say that you now have your entities in your cache friendly array. But then you think that you want to introduce components to your entities, so you start writing a whole data-oriented Entity-Component-System (ECS).

For a small indie game or similar, just making sure to do things in a basic, cache friendly way is more than enough. Just avoiding the separate heap allocations is a good start. There's no need to go all the way to an ECS system. Personally, I think ECS is often an over-engineered solution that makes gameplay code both harder and less fun to write.

Try to get a feeling for when the code design decisions you make are hurting your creative flow. Find a good middle-ground between creativity, code tidiness and performance. "Good enough" is very subjective, but you must look at what your goals are and if your technical decisions are helping or hurting your trajectory towards those goals. If your goal is to ship a game, then make sure to keep an eye on your technical decisions, so that they don't add too much friction to your creative programming flow.

## Thanks for reading!

In my book _Understanding the Odin Programming Language_ I have a chapter that talks about these ideas in more details. Read a sample of the book here: https://odinbook.com/

If you're interested in more gamedev content, then check out my YouTube channel: https://www.youtube.com/@karl_zylinski

If you wish to support me, then you can become a patron: https://www.patreon.com/karl_zylinski

Have a great day!
/Karl Zylinski