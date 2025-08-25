---
title: "Understand the Temporary Allocator; Understand arenas"
date: 2025-08-25T20:45:00+02:00

cover:
  image: "/temporary-allocator/cover.png"
---

Odin is a manually memory managed language. Beginners of such languages are often afraid of memory leaks, meaning that you forget to deallocate memory.

The first remedy for this is to [use the tracking allocator](https://odin-lang.org/docs/overview/#tracking-allocator). It will warn you about memory you forgot to deallocate. Nifty!

But you can also think along these lines: Can I simplify things a bit, so I don't have to think about the deallocation?

What you can do is to try to use the temporary allocator whenever possible. Let's look at a real-world example.

In my game [CAT & ONION](https://zylinski.itch.io/cat-and-onion) I draw a UI of all the actions in the game. You see this UI in the bottom left of this screenshot:

![A screenshot of a video game with a UI displaying some possible actions in the bottom left corner](/temporary-allocator/cat_and_onion.png)

This list of possible actions is returned by the procedure below. It returns a dynamic array of `PlayerAction` objects. This array contains all all the possible actions of the player. There are three actions you can always do, and then an unknown number of actions available based on what you have in your inventory. The calls to `append` will cause dynamic allocations when the dynamic array `actions` needs to grow. This first version does not use the temporary allocator, we'll add in the temporary allocator shortly.

```go
get_all_player_actions :: proc() -> [dynamic]PlayerAction {
	actions: [dynamic]PlayerAction

	append(&actions, PlayerAction {
		type = .Inspect,
	})

	append(&actions, PlayerAction {
		type = .Talk,
	})

	append(&actions, PlayerAction {
		type = .Pickup,
	})

	for i in g_mem.inventory.items {
		append(&actions, PlayerAction {
			type = .UseItem,
			item = i.item,
			added_at = i.added_at,
			item_original_screen_pos = i.original_screen_pos,
		})
	}

	return actions
}
```

Here is some code that runs the procedure and then displays all the actions on the screen:

```go
actions := get_all_player_actions()

for a in actions {
	draw_action_icon(a)
}

delete(actions)
```

Note how we loop over the list of actions and draw an icon for each. After the loop I call `delete(actions)`, because the calls to `append` within `get_all_player_actions` caused dynamic allocations. Forgetting the call to `delete` would leak the allocated memory. Since this UI drawing code runs each frame of the game, it would cause the game's memory usage to continuously grow: A classic memory leak. The tracking allocator would catch these leaks. But it would be nice if I never had to worry about this at all, tracking allocator or no tracking allocator.

That's where the temporary allocator comes in.

In the game I actually only need the list of actions to be around for drawing or doing some logic checks. With that in mind, the _actual_ code that I use in CAT & ONION is the following. The only difference is the first line of the procedure: When I create the `actions` array, I tell it to use the temporary allocator. 

```go
get_all_player_actions :: proc() -> [dynamic]PlayerAction {
	actions := make([dynamic]PlayerAction, context.temp_allocator)

	append(&actions, PlayerAction {
		type = .Inspect,
	})

	append(&actions, PlayerAction {
		type = .Talk,
	})

	append(&actions, PlayerAction {
		type = .Pickup,
	})

	for i in g_mem.inventory.items {
		append(&actions, PlayerAction {
			type = .UseItem,
			item = i.item,
			added_at = i.added_at,
			item_original_screen_pos = i.original_screen_pos,
		})
	}

	return actions
}
```

> The line `actions := make([dynamic]PlayerAction, context.temp_allocator)` does not cause any allocation. It just returns a dynamic array with its internal `allocator` field set to `context.temp_allocator`. The allocations happen later, as `append` runs. The dynamic array will need to grow at least once (it has the initial size of zero). It will then use the temporary allocator we fed into `make`.
> This "internal `allocator` field" exists on the struct `Raw_Dynamic_Array`, which is the type that a dynamic array uses "behind the scenes". You can see that type here: https://pkg.odin-lang.org/base/runtime/#Raw_Dynamic_Array

Everything that goes into the temporary allocator is valid until you clear it. You clear it by placing `free_all(context.temp_allocator)` somewhere in your program. In games that often happens at the "end of each frame":

```go
main :: proc() {
	for game_should_run {
		update()
		draw()
		free_all(context.temp_allocator)
	}
}
```

Now, when drawing the icons, I can skip the `delete(actions)` line:

```go
actions := get_all_player_actions()

for a in actions {
	draw_action_icon(a)
}
```

The `actions` array will thus be valid until `free_all(context.temp_allocator)` happens, meaning _until the end of frame_.

## Your first arena allocator

You may have heard about arena allocators. You may be a bit confused of when it is a good idea to use them.

An arena allocator has the property that all the allocations into it shares a common _lifetime_. This means that they will all get deallocated at the same time.

If you look back at the code above, then you see that everything on the temporary allocator has the _lifetime of a single frame_, because we clear it at the end of the frame.

In other words: The temporary allocator is an arena allocator. It has a predetermined use-case (temporary things), but you chose what temporary means (when do you clear it? What is the lifetime?)

In video games the line `free_all(context.temp_allocator)` is very easy to place: At the end of the frame. It's an unusually easy-to-spot lifetime that is shared by an abundance of allocations. It's a blessed thing about video games. Need a list of player actions? Just return a temporary allocated array. Need to draw a formatted string on the screen each frame? Just temporary allocate it and draw. Need to store an array of intermediate calculations? Allocate the array using the temporary allocator.

For non-games the line `free_all(context.temp_allocator)` is sometimes harder to place, because there may not be an obvious "frame loop". It's OK to simply not use the temporary allocator in that case! Use a tool when it helps you. But remember: The temporary allocator is just a special arena allocator. You can make your own arena allocators when you realize that a few things share a common lifetime (read more about creating arenas [in a previous blog post](https://zylinski.se/posts/dynamic-arrays-and-arenas/)). However, don't try to force things into an arena. Just use the default heap allocator when in doubt. With time you'll notice things of common lifetime, and at that moment you can experiment with custom arenas.

If you enjoyed this post and want to learn more about Odin, then check out my eBook "Understanding the Odin Programming Language": https://odinbook.com

Have a nice day!
/Karl Zylinski

