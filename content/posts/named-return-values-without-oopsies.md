---
title: "Named Return Values Without Oopsies"
date: 2026-09-01T22:35:00+01:00
draft: true
---

I have devised a new way of working with named return values in Odin. Named return values tend to make me a bit uneasy. This new way removes that.

Let's first talk about what named return values are and how they can be weird to use.

Named return values look like this:

```go
load_audio_stream_from_file :: proc(
	filename: string,
) -> (
	stream: Audio_Stream,
	ok: bool,
) #optional_ok {
	// blabla
}
```

> This is a procedure signature from my 2D game creation library [Karl2D](https://karl2d.com). It's for loading a stream of audio, such as a music file.

`stream` and `ok` are the named return values. They can be assigned to like any variable. They are zero-initialized by default, like any variable.

With named return values you have two ways to return: ordinary, and ~naked~.

Ordinary is just `return some_stream, true`.

Naked returns look like this:

```go
stream = make_some_stream()
ok = true
return
```

Naked returns send back whatever is in `stream` and `ok` when `return` happens. I don't like it! It's easy to make a mistake. Something invalid might be in `stream` when the `return` happens.

However, `load_audio_stream_from_file` is a long procedure. I had a lot of error states where I needed to do `return AUDIO_STREAM_NONE, false`. For _that reason_ I added named return values so I could just write `return` and send back the zero value of both `stream` and `ok`. `AUDIO_STREAM_NONE` was in this case the same as the zero value. In other words: I just wanted naked `return`s for a single thing: returning everything zeroed, to signal that an error had happened.

> Named return values also allow you to use `or_return`, which many like.

Now the problem with adding a named return value to my proc `load_audio_stream_from_file` was that I had this code at the bottom:

```go
stream, stream_add_err := hm.add(&s.audio_streams, stream_data)

if stream_add_err != nil {
	// CLEANUP CODE GOES HERE
	return
}

return stream, true
```

Note that `stream` is the same name as the named return value I just added. They would now shadow each other, which won't compile. I could change that `:=` for a `=` and just write to the `stream` return value directly. But I _do not_ want to assign to the return value. Why? If the `stream_add_err != nil` case is hit, then I would have to unset `stream` again before returning. Yuck! Another likely scenario is that I just forget to unset `stream`, which would return some value I didn't intend to, since the "I just hit an error"-usage of the naked `return` should return everything zeroed.

> I would also have to do something like
> ```go
> stream_add_err: runtime.Allocator_Error
> stream, stream_add_err = hm.add(&s.audio_streams, stream_data)
> ```
> 🫠

Here we see how the combo of named return values and naked returns can create bugs. Bugs where you return things you didn't intend to.

Here comes my fix: Whenever I use named return values, give them a `_` prefix. This underscore is a marker that yells "don't assign to me!!". Previously people could assign to it by mistake. But the name collisions could also lead them to attempting to be smart, which is the most dangerous idea a programmer can have.

In other words, the procedure signature becomes this:

```go
load_audio_stream_from_file :: proc(
	filename: string,
) -> (
	_stream: Audio_Stream,
	_ok: bool,
) #optional_ok {
	// blabla
}
```

I can still use my naked return, which just means "return everything zeroed", which signals the error state. But the underscore is a good mental note to myself to not try to do any clever things.

> You may wonder why I split my procedure across so many lines. It's because it would be more than 100 characters wide if written on a single line. 100 is my maximum. When splitting a procedure into several lines, I write both the list of parameters and return values the same way that I write the fields of a struct.

Thanks for reading! Check out my library [Karl2D](https://karl2d.com) and my book [Understanding the Odin Programming Language](https://odinbook.com).

/Karl Zylinski