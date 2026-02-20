---
title: "Audio in Karl2D: Software mixing, OS APIs and general design"
date: 2026-02-20T22:04:00+02:00

cover:
  image: "/audio-pontus.jpg"
---

As I've been making my own game creation library [Karl2D](https://github.com/karl-zylinski/karl2d) I've noticed something about audio: While many people are fine with writing their own rendering, they still go for some library when implementing sound. For example, you might end up using the library [miniaudio](https://miniaud.io/).

I decided to try to implement my own audio in Karl2D, and so far I'd say it has been easier and more straight-forward than rendering. It has also been quite fun! I think people who make games using my library will appreciate being able to debug windowing, input, rendering and audio all in a single library.

## The balance between low-levelness and choosing audio APIs

The audio libraries that operating systems expose have a varying level of high-levelness. Meaning that some just give you a buffer to write some audio into and some handle everything for you, including mixing and audio effects.

On most operating systems there are high-level and a low-level audio APIs. If you decide to use these high level APIs then they will often take care of audio effects and mixing. When comparing high level audio APIs from different platforms, then they tend to do things in slightly different ways. Because of this, your audio sound slightly different on different platforms.

So instead of all that I decided to use the most basic APIs and do the mixing in my own code. This is known as writing your own "software mixer".

The code I ended up writing can be summarized as doing the following:

- Go over all playing sounds
- Mix them into a "mix buffer"
- Feed the mixed buffer to a platform-specific backend
- The backend sends the sound off to the OS API

Using this method, I have so far managed to add audio support for Windows, Web and Linux.

> Mac coming soon!

For each platform I needed to choose an audio API. For Windows I went with `waveOut`, for Web I use `Web Audio` and for Linux I use `ALSA`.

I'll probably add WASAPI on Windows at some later point, as it can push the latency down (less "audio lag"). But waveOut is easier to get started with. For Linux, there are a lot of audio servers and systems. My initial code just supports `ALSA`, I will see if I need to add more on Linux later.

## Audio state and mixing

I added the following fields to the main state object of Karl2D (accessible as the global `s` inside `karl2d.odin`)

```go
audio_backend: Audio_Backend_Interface,
audio_backend_state: rawptr,

sound_data: hm.Dynamic_Handle_Map(Sound_Data, Sound_Data_Handle),
sound_instances: hm.Dynamic_Handle_Map(Sound_Instance, Sound),

// Sounds that have been started as because `play_sound` was called.
playing_sounds: [dynamic]Playing_Sound,

// 1 megabyte is arbitrarily chosen.
mix_buffer: [1*mem.Megabyte]Audio_Sample,

// Where the mixer currently is in the mix buffer.
mix_buffer_offset: int,
```

The `audio_backend` implements a simple interface for feeding mixed audio samples to the platform's audio API. The interface definition is a struct that looks like this:

```go
Audio_Backend_Interface :: struct #all_or_none {
	state_size: proc() -> int,
	init: proc(state: rawptr, allocator: runtime.Allocator),
	shutdown: proc(),
	set_internal_state: proc(state: rawptr),

	feed: proc(samples: []Audio_Sample),
	remaining_samples: proc() -> int,
}
```

As you can see, this is a very simple interface. Since we're doing software mixing, all that the backend needs to do is receive mixed audio samples and send them off to the audio API we've chosen the current platform. So there is one backend for `waveOut`, one for `ALSA` and one for `Web Audio`.

`mix_buffer` is a big array of audio samples, where and `Audio_Sample` is defined as `Audio_Sample :: [2]f32`. It has two channels (left and right) and each sample is stored as a floating point value. This is the format used by the mixer and loaded stereo sounds.

> You may ask: "Don't you need more channels. What about mixing surround sound and such?". I don't see surround in 2D game very often. So I'm starting without it. Those who play 2D games on surround speaker systems tend to just "mirror" the stereo signal to the rear speakers.
>
> This is also the format used for loaded WAV files. For mono sounds I'll probably have to add another format, at which point the loaded WAV might need an abstraction of the data format to support both samples that are just `[2]f32` and `f32`. The alternative is clone the mono data to both channels when you load the sound. But that is wasteful. 

Loading of sounds is done by `load_sound_from_file` and `load_sound_from_bytes` in [karl2d.odin](https://github.com/karl-zylinski/karl2d/blob/master/karl2d.odin). The `load_sound_from_bytes` procedure implements some homemade WAV loading based on what I read in the [WAV wikipedia article](https://en.wikipedia.org/wiki/WAV#WAV_file_header). The samples of each loaded sound is stored as `Sound_Data` structs:

```go
Sound_Data :: struct {
	handle: Sound_Data_Handle,
	samples: []Audio_Sample,
	sample_rate: int,

	// When a Sound_Instance is destroyed, we check if this reaches zero. If it does, then the
	// Sound_Data and its samples slice are also destroyed/freed.
	instances: int,
}
```

and then there is a concept of a `Sound_Instance`, which is a reference to the data plus some information about how to play that data:

```go
Sound_Instance :: struct {
	handle: Sound,
	sound_data_handle: Sound_Data_Handle,
	volume: f32,
	target_volume: f32,
	pan: f32,
	target_pan: f32,
	pitch: f32,
	target_pitch: f32,
}
```

I have this split so that I can create multiple sound instances that all reference the same `Sound_Data`. This way I can play a single sound several times simultaneously without having to duplicate the data. For example, I can make two instances of a sound and play them at two different pitches simultaneously. This is good for creating variation without allocating extra memory.

## Playing sounds and mixing them

I currently do the mixing on the main thread. I might add a mixer thread at some point. However, starting on the main thread is good for focusing on the problem rather than getting lost in concurrency issues.

The `play_sound` procedure just does this:
```go
play_sound :: proc(snd: Sound, loop := false) {
	append(
		&s.playing_sounds,
		Playing_Sound {
			sound = snd,
			loop = loop,
		},
	)
}
```

The "playing sounds" will be processed by the mixer procedure. That procedure is rather big; see [appendix A](#appendix-a-update_audio_mixer) for the complete procedure, or for `update_audio_mixer` in [karl2d.odin](https://github.com/karl-zylinski/karl2d/blob/master/karl2d.odin) to see the whole thing. Here I will go through the important parts.

The abstract structure of `update_audio_mixer` is like this:
```go
update_audio_mixer :: proc() {
	if ab.remaining_samples() > (3 * AUDIO_MIX_CHUNK_SIZE)/2 {
		return
	}

	for &ps in s.playing_sounds {
		for samp_idx in 0..<AUDIO_MIX_CHUNK_SIZE {
			// add samples from `ps.samples` into `s.mix_buffer` and advance `ps.offset`
		}
	}

	ab.feed(s.mix_buffer[s.mix_buffer_offset:AUDIO_MIX_CHUNK_SIZE])

	// advance s.mix_buffer_offset
}
```

Here `ab` is the audio backend. The if statement at the top of the procedure means that we only run the mixer if the audio backend is "close to starvation". This means that it will soon run out of samples, in our case it means that fewer than `1.5*AUDIO_MIX_CHUNK_SIZE` number of samples remains to be played by the backend. This `AUDIO_MIX_CHUNK_SIZE` is the number of samples we mix when the mixer does run. This way, a maximum of `2.5*AUDIO_MIX_CHUNK_SIZE` number of samples will ever be in the mixer buffer. At a sample rate of 44100 samples/second and a `AUDIO_MIX_CHUNK_SIZE` of `1400` this means that we have a maximum latency of `(1400*2.5)/44100=0.079 seconds`, or `79 ms`. That may be a bit high. We could make the chunk size smaller or use a smaller value than `1.5*AUDIO_MIX_CHUNK_SIZE` in the if statement. In any case, these are things I can easily tweak.

> Perhaps I should have different chunk sizes for different backends. Some of them have less latency than others. The important thing is to never let the backend run dry, because then the audio just stops. 

As we saw before, the mix buffer is `mix_buffer: [1*mem.Megabyte]Audio_Sample`. We use this buffer in a circular fashion: We mix in `AUDIO_MIX_CHUNK_SIZE` into it and send that off to the backend. If we reach the end of mix buffer, then we just start over from the beginning. Since the buffer is big, we can be sure that the samples we've sent off to the backend have been played already, before they are overwritten again. This way we do not need to do any extra allocation when sending the samples off to the backend: We just send it a slice that looks into a section of this big circular buffer.

> Using a whole megabyte is very overkill. I'll adjust it at some point.

Writing a software mixer may sound scary, but it really isn't. It's mainly about adding the samples from the playing sounds into the mix buffer. I have two procedures for doing this: `add` and `add_interpolate`. The first one is used when the number of samples per second matches the mixer buffer. In my case I use 44100 samples per second for the mixer buffer. If the playing sound also uses that then the mixing just becomes adding the samples into the mixer buffer. If the sample rate of the playing sound is lower, for example 22050 samples/second, then we need to interpolate to "fill in the gaps". You can get by with just simple linear interpolation. The interpolation code can be a bit tricky to get right. What I do is calculate the ratio between the mixer sample rate and the playing sound's sample rate:

```go
samples_per_mixer_sample := (pitch*f32(data.sample_rate))/f32(AUDIO_MIX_SAMPLE_RATE)
```

I feed that value to `add_interpolate` and use it to figure out which source samples are the previous and current ones, and interpolate between them.

If you want to do this yourself, then just start with loading some 44100 Hz sound and use a 44100 Hz mixer buffer and just do the sample addition. Worry about the interpolation when the simple stuff works.

> I started even simpler: I generated a sine wave and made that one play correctly. You can see my sine wave generation code in the [audio example](https://github.com/karl-zylinski/karl2d/blob/master/examples/audio/audio.odin).

## Interpolating volume and pan changes to avoid audio clicks

If you suddenly change the volume of a sound or pan it quickly, then the audio may "click" or "pop". This is due to discontinuities introduced in the audio wave. Loud sounds happen when the audio data quickly changes from one sample value to another. So if you change volume from 0.2 to 1 (given a range of 0-1), then a loud click will happen: The speaker wants to move very quickly from one state to another.

To avoid this, don't set the volume and pan directly. Instead, set a target volume and target pan on each sound. Then quickly interpolate towards that value. As long as you interpolate within a few tens of milliseconds, it should be OK.

When mixing each playing sound I first calculate what start and end state for the current "mixing pass" should be:

> By "mixing pass" I mean when the mixer mixes another `AUDIO_MIX_CHUNK_SIZE` number of samples into the mixer buffer.

```go
move_towards :: proc(current: f32, target: f32, delta: f32) -> f32 {
	if abs(target - current) < delta {
		return target
	}

	dir := math.sign(target - current)
	return current + dir * delta
}


volume_start := clamp(inst.volume, 0, 1)
volume_end := clamp(move_towards(inst.volume, inst.target_volume, adjust_parameter_delta), 0, 1)
```

Here `adjust_parameter_delta` is something like `44100.0 * 0.3`, i.e. I want it to take 0.3 second to go from volume `0` to `1`. Experiment to find values that avoid clicks.

I feed these `volume_start` and `volume_end` sounds into my addition procedures. In there I lerp between them 

```go
add :: proc(
	dest: []Audio_Sample,
	source: []Audio_Sample,
	num: int,
	volume_start: f32,
	volume_end: f32,
	pan_start: [2]f32,
	pan_end: [2]f32,
) -> int {
	to_write := num

	if to_write > len(source) {
		to_write = len(source)
	}
	
	for samp_idx in 0..<to_write {
		t := f32(samp_idx) / f32(to_write)
		volume := math.lerp(volume_start, volume_end, t)
		pan := linalg.lerp(pan_start, pan_end, t)
		dest[samp_idx] += pan * source[samp_idx] * volume
	}

	return to_write
}
```

```go
add(
	s.mix_buffer[s.mix_buffer_offset:],
	data.samples[ps.offset:],
	AUDIO_MIX_CHUNK_SIZE,
	volume_start,
	volume_end,
	pan_stereo_start,
	pan_stereo_end,
)
```

The reason for lerping between `volume_start` and `volume_end` is that I mix `AUDIO_MIX_CHUNK_SIZE` of samples in one go. If I just hop to a single volume or pan value for all those samples, then it will still click since a chunk is over a thousand samples.

Note that `volume_end` is not the same as the target volume of sound. It's just the volume that the sound should have after an additional `AUDIO_MIX_CHUNK_SIZE` number of samples have been mixed. The interpolation can happen over several mixer passes.

> How many mixer passes does it take to go from 0 to 1 if we use the `adjust_parameter_delta` with value `44100 * 0.3`? Well, `44100*0.3` is `13230` samples. We mix 1400 samples in one chunk. `13230/1400=9.45`. So it takes `9.45`, or `10` mixing passes to go from volume `0` to `1`.

## Feeding the mixed audio to the backend

At the end of the mixing pass, we take the `AUDIO_MIX_CHUNK_SIZE` number of samples just mixed into the mix buffer and feed them to the backend.

Thanks to the software mixing, the backend is very simple. When creating the backend, I just open the audio API for writing using the correct sample rate and such. Then there is a procedure for feeding in the samples. An example of this simplicity: The waveOut backend used on Windows is currently just 125 lines of code. That's so little that I can just put the whole backend here:
```go
#+build windows
#+vet explicit-allocators
#+private file
package karl2d

@(private="package")
AUDIO_BACKEND_WAVEOUT :: Audio_Backend_Interface {
	state_size = waveout_state_size,
	init = waveout_init,
	shutdown = waveout_shutdown,
	set_internal_state = waveout_set_internal_state,

	feed = waveout_feed,
	remaining_samples = waveout_remaining_samples,
}

import "base:runtime"
import "log"
import win32 "core:sys/windows"
import "core:time"
import "core:slice"

Waveout_State :: struct {
	device: win32.HWAVEOUT,
	allocator: runtime.Allocator,
	headers: [32]win32.WAVEHDR,
	cur_header: int,
	submitted_samples: int,
}

waveout_state_size :: proc() -> int {
	return size_of(Waveout_State)
}

s: ^Waveout_State

waveout_init :: proc(state: rawptr, allocator: runtime.Allocator) {
	assert(state != nil)
	s = (^Waveout_State)(state)
	s.allocator = allocator
	log.debug("Init audio backend waveout")

	// Added constant missing in bindings:
	// KSDATAFORMAT_SUBTYPE_IEEE_FLOAT GUID: 00000003-0000-0010-8000-00aa00389b71
	KSDATAFORMAT_SUBTYPE_IEEE_FLOAT :: win32.GUID{0x00000003, 0x0000, 0x0010, {0x80, 0x00, 0x00, 0xaa, 0x00, 0x38, 0x9b, 0x71}}

	format := win32.WAVEFORMATEXTENSIBLE {
		Format = {
			nSamplesPerSec = 44100,
			wBitsPerSample = 32,
			nChannels = 2,
			wFormatTag = win32.WAVE_FORMAT_EXTENSIBLE,
			cbSize = size_of(win32.WAVEFORMATEXTENSIBLE) - size_of(win32.WAVEFORMATEX),
		},
		Samples = {
			wValidBitsPerSample = 32,
		},
		dwChannelMask = { .FRONT_LEFT, .FRONT_RIGHT },
		SubFormat = KSDATAFORMAT_SUBTYPE_IEEE_FLOAT,
	}

	format.nBlockAlign = (format.wBitsPerSample * format.nChannels) / 8 // see nBlockAlign docs
	format.nAvgBytesPerSec = (u32(format.wBitsPerSample * format.nChannels) * format.nSamplesPerSec) / 8

	ch(win32.waveOutOpen(
		&s.device,
		win32.WAVE_MAPPER,
		&format,
		0,
		0,
		win32.CALLBACK_NULL,
	))
}

ch :: proc(mr: win32.MMRESULT, loc := #caller_location) -> win32.MMRESULT {
	if mr == 0 {
		return mr
	}

	log.errorf("waveout error. Error code: %v", u32(mr), location = loc)
	return mr
}

waveout_shutdown :: proc() {
	log.debug("Shutdown audio backend waveout")
	win32.waveOutClose(s.device)
}

waveout_set_internal_state :: proc(state: rawptr) {
	assert(state != nil)
	s = (^Waveout_State)(state)
}

waveout_feed :: proc(samples: []Audio_Sample) {
	h := &s.headers[s.cur_header]

	for win32.waveOutUnprepareHeader(s.device, h, size_of(win32.WAVEHDR)) == win32.WAVERR_STILLPLAYING {
		time.sleep(1 * time.Millisecond)
	}

	byte_samples := slice.reinterpret([]u8, samples)

	h^ = {
		dwBufferLength = u32(len(byte_samples)),
		lpData = raw_data(byte_samples),
	}

	win32.waveOutPrepareHeader(s.device, h, size_of(win32.WAVEHDR))
	win32.waveOutWrite(s.device, h, size_of(win32.WAVEHDR))

	s.submitted_samples += len(samples)
	s.cur_header += 1

	if s.cur_header >= len(s.headers) {
		s.cur_header = 0
	}
}

waveout_remaining_samples :: proc() -> int {
	t := win32.MMTIME {
		wType = .TIME_SAMPLES,
	}
	win32.waveOutGetPosition(s.device, &t, size_of(win32.MMTIME))
	return s.submitted_samples - int(t.u.sample)
}
```

## Web audio

For the web the Odin part of the backend looks like this:

```go
#+build js
#+vet explicit-allocators
#+private file
package karl2d

@(private="package")
AUDIO_BACKEND_WEB_AUDIO :: Audio_Backend_Interface {
	state_size = web_audio_state_size,
	init = web_audio_init,
	shutdown = web_audio_shutdown,
	set_internal_state = web_audio_set_internal_state,
	feed = web_audio_feed,
	remaining_samples = web_audio_remaining_samples,
}

import "base:runtime"
import "core:slice"

foreign import karl2d_web_audio "karl2d_web_audio"

// The `js_` prefix is there to just avoid clashes with the procs in this file.
@(default_calling_convention="contextless")
foreign karl2d_web_audio {
	@(link_name="web_audio_init")
	js_web_audio_init :: proc() ---
	@(link_name="web_audio_shutdown")
	js_web_audio_shutdown :: proc() ---
	@(link_name="web_audio_feed")
	js_web_audio_feed :: proc(samples: []f32) ---
	@(link_name="web_audio_remaining_samples")
	js_web_audio_remaining_samples :: proc() -> int ---
}

web_audio_state_size :: proc() -> int {
	return 0
}

web_audio_init :: proc(state: rawptr, allocator: runtime.Allocator) {
	js_web_audio_init()
}

web_audio_shutdown :: proc() {
	js_web_audio_shutdown()
}

web_audio_set_internal_state :: proc(state: rawptr) {
	// No hot reload on web.
}

web_audio_feed :: proc(samples: []Audio_Sample) {
	// The JS backend just sees an array of f32. But it knows that they are interleaved Left & Right
	js_web_audio_feed(slice.reinterpret([]f32, samples))
}

web_audio_remaining_samples :: proc() -> int {
	return js_web_audio_remaining_samples()
}
```

This is even simpler than the waveOut code. However, as you see, there is some bindings in there (the `foreign karl2d_web_audio {` stuff). Those are bindings to JavaScript code I've written. That code can be seen in [`audio_backend_web_audio.js`](https://github.com/karl-zylinski/karl2d/blob/master/audio_backend_web_audio.js) and [`audio_backend_web_audio_processor.js`](https://github.com/karl-zylinski/karl2d/blob/master/audio_backend_web_audio_processor.js). As with Windows, its mostly about opening some audio system for writing samples. It's a bit more complicated on web due to it using some complicated "audio worklet" system. Also, I had to modify the `index.html` file that loads the WASM code so that it loads in this extra JS code.

> The `index.html` file loads the `odin.js` runtime, my extra audio stuff and then runs the compiled WASM code.

## Linux ALSA, and avoiding LLM slop

I've also made a Linux ALSA backend that turned out quite simple: [`audio_backend_alsa.odin`](https://github.com/karl-zylinski/karl2d/blob/master/audio_backend_alsa.odin). For that backend, I tried having GitHub copilot help me write the basics. I haven't used AI/LLM stuff much before. It gave a good foundation, including creating the ALSA bindings. But I made sure to understand every binding it added: I looked at the documentation of the ALSA procedures and made sure they were what I needed. I had to edit the code it wrote a lot, removing over-complicated stuff and stuff that felt off. The result is 146 lines of ALSA backend and 49 lines of ALSA bindings.

Use LLMs with great care. Use them for contained things where you can understand all the things without being overwhelmed. The moment you accept code into your code base without understanding it, then you are on a slippery slope towards _slop_. If you can't fight that urge, then just stop using such tools. It will ruin your code since you will lose control over it, making it unmaintainable. If you accept slop then very soon the only way to maintain the repository is to go 100% vibe code. And then you have no idea what you are making anymore. The craft is then dead.

## What remains?

I still need to add streaming of music. I might support OGG Vorbis first. For that I might use stb_vorbis.

> While I don't like using a library for the audio backend, using a library to load files is fine. The music loading library is a "leaf node": You can easily replace it without any problems and it also does a very contained task.

Also, I might want to add a thread for the mixing. But that is a future me problem.

Thanks for reading!

If you want to support the creation of Karl2D, then support me by sponsoring me on [GitHub](https://github.com/sponsors/karl-zylinski) or [Patreon](https://www.patreon.com/karl_zylinski).

Further reading on this topic: [Writing a Low-Level Sound System — You Can Do It!](https://ruby0x1.github.io/machinery_blog_archive/post/writing-a-low-level-sound-system/index.html) on the Our Machinery blog archive.

Have a nice day!
/Karl

## Appendix A: `update_audio_mixer`

```go
// Update the audio mixer and feed more audio data into the audio backend. This is done
// automatically when `update` runs, so you normally don't need to call this manually.
//
// This procedure implements a custom software audio mixer. The backend is just fed the resulting
// mix. Therefore, you can see everything regarding how audio is processed in this procedure.
//
// Will only run if the audio backend is running low on audio data.
update_audio_mixer :: proc() {
	// If the sample rate of the backend is 44100 samples/second and AUDIO_MIX_CHUNK_SIZE is 1400
	// samples, then this procedure will only run roughly 44100/1400 = 31 times per second. This
	// gives a latency of up to (1.5 * (44100/1400)) = 47 milliseconds. Is it too big, or too small?
	// Perhaps we can use more low latency backends to push it down. Perhaps the backend should
	// control AUDIO_MIX_CHUNK_SIZE based on how low latency it can give us without stalling?
	if ab.remaining_samples() > (3 * AUDIO_MIX_CHUNK_SIZE)/2 {
		return
	}
	
	// We are going to go past the end of the mix_buffer, so just hop to the start instead. It's
	// 1 megabyte big, so hopping over a few bytes at the end is OK.
	if (s.mix_buffer_offset + AUDIO_MIX_CHUNK_SIZE) > len(s.mix_buffer) {
		s.mix_buffer_offset = 0
	}

	// A slice of the mixed samples we are going to output.
	out := s.mix_buffer[s.mix_buffer_offset:s.mix_buffer_offset + AUDIO_MIX_CHUNK_SIZE]
	
	// Zero out old mixed data from buffer (the buffer is "circular", there may be old stuff in
	// the `out` slice).
	slice.zero(out)

	// For usage when the sample rates of the playing sound and the mixer match.
	add :: proc(
		dest: []Audio_Sample,
		source: []Audio_Sample,
		num: int,
		volume_start: f32,
		volume_end: f32,
		pan_start: [2]f32,
		pan_end: [2]f32,
	) -> int {
		to_write := num

		if to_write > len(source) {
			to_write = len(source)
		}
		
		for samp_idx in 0..<to_write {
			t := f32(samp_idx) / f32(to_write)
			volume := math.lerp(volume_start, volume_end, t)
			pan := linalg.lerp(pan_start, pan_end, t)
			dest[samp_idx] += pan * source[samp_idx] * volume
		}

		return to_write
	}

	// For usage when the sample rates don't match. Needs a `dest_source_ratio` parameter that tells
	// us how the sample ratios relate. It's used for getting from indices from dest sample space
	// to source sample space.
	add_interpolate :: proc(
		dest: []Audio_Sample,
		source: []Audio_Sample,
		source_offset: f32,
		num_dest: int,
		dest_source_ratio: f32,
		volume_start: f32,
		volume_end: f32,
		pan_start: [2]f32,
		pan_end: [2]f32,
	) -> int {
		
		dest_idx: int
		for ; dest_idx < num_dest; dest_idx += 1 {
			src_pos := source_offset + f32(dest_idx) * dest_source_ratio
			src_idx := int(src_pos)
			
			if src_idx >= len(source) {
				break
			}

			src_next := min(src_idx + 1, len(source) - 1)
			frac := src_pos - f32(src_idx)

			prev_val := source[src_idx]
			cur_val := source[src_next]

			t := f32(dest_idx) / f32(num_dest)
			volume := math.lerp(volume_start, volume_end, t)
			pan := linalg.lerp(pan_start, pan_end, t)

			dest[dest_idx] += pan * linalg.lerp(prev_val, cur_val, frac) * volume
		}

		return dest_idx
	}

	for idx := 0; idx < len(s.playing_sounds); idx += 1 {
		ps := &s.playing_sounds[idx]
		inst := hm.get(&s.sound_instances, ps.sound)

		if inst == nil {
			log.error("Trying to play destroyed sound")
			unordered_remove(&s.playing_sounds, idx)
			idx -= 1
			continue
		}

		data := hm.get(&s.sound_data, inst.sound_data_handle)

		if data == nil {
			log.error("Trying to play sound with destroyed data")
			unordered_remove(&s.playing_sounds, idx)
			idx -= 1
			continue
		}

		// Before we get to the mixing we smoothly adjust pitch, volume and pan. We do this to avoid
		// clicks in the audio. The clicks happen because abrupt changes cause discontinuities in
		// the audio waveform. Understand: Sound does not happen because the waveform has a high
		// value, it happens because there is a sudden change in the waveform. Bigger change, bigger
		// sound.

		calc_adjust_parameter_delta :: proc(sample_rate: int, pitch: f32) -> f32 {
			RAMP_TIME :: 0.03
			ramp_samples := RAMP_TIME * f32(sample_rate) * pitch
			return AUDIO_MIX_CHUNK_SIZE / ramp_samples
		}

		move_towards :: proc(current: f32, target: f32, delta: f32) -> f32 {
			if abs(target - current) < delta {
				return target
			}

			dir := math.sign(target - current)
			return current + dir * delta
		}

		// We get the delta twice because we first need to move the pitch towards its target.
		adjust_parameter_delta := calc_adjust_parameter_delta(data.sample_rate, max(inst.pitch, 0.01))
		inst.pitch = max(move_towards(inst.pitch, inst.target_pitch, adjust_parameter_delta), 0.01)
		pitch := inst.pitch
		adjust_parameter_delta = calc_adjust_parameter_delta(data.sample_rate, pitch)

		// We can't just use the `volume_end` value for the volume. We are going to mix in
		// `AUDIO_MIX_CHUNK_SIZE` number of samples. We'd still get clicks in the sound if we hopped
		// to the ending volume. Instead, we calculate what the first sample should use and what
		// the last one should use. Then we feed those into the `add`/`add_interpolate` procedures.
		// It will lerp across the range as it is mixing in the samples.

		volume_start := clamp(inst.volume, 0, 1)
		volume_end := clamp(move_towards(inst.volume, inst.target_volume, adjust_parameter_delta), 0, 1)
		inst.volume = volume_end

		if volume_start == volume_end && volume_end == 0 {
			continue
		}
		
		pan_start := clamp(inst.pan, -1, 1)
		pan_end := clamp(move_towards(inst.pan, inst.target_pan, adjust_parameter_delta), -1, 1)
		inst.pan = pan_end
		
		// Use cos/sine to get a constant-power audio curve. This means that the sound won't get
		// quieter in the middle, but will instead just pan.
		pan_stereo_start := [2]f32 {
			math.cos((pan_start + 1) * math.PI / 4),
			math.sin((pan_start + 1) * math.PI / 4),
		}

		pan_stereo_end := [2]f32 {
			math.cos((pan_end + 1) * math.PI / 4),
			math.sin((pan_end + 1) * math.PI / 4),
		}

		interpolate := data.sample_rate != AUDIO_MIX_SAMPLE_RATE || pitch != 1
		num_mixed: int
		
		if interpolate {
			samples_per_mixer_sample := (pitch*f32(data.sample_rate))/f32(AUDIO_MIX_SAMPLE_RATE)

			num_mixed = add_interpolate(
				s.mix_buffer[s.mix_buffer_offset:],
				data.samples[ps.offset:],
				ps.offset_fraction,
				AUDIO_MIX_CHUNK_SIZE,
				samples_per_mixer_sample,
				volume_start,
				volume_end,
				pan_stereo_start,
				pan_stereo_end,
			)
			
			num_mixed_f32 := f32(num_mixed) * samples_per_mixer_sample
			fraction_advance := ps.offset_fraction + num_mixed_f32

			// The fraction advance may become larger than 1, in which case the offset needs to eat
			// the integer part.
			ps.offset += int(fraction_advance)
			
			ps.offset_fraction = linalg.fract(fraction_advance)
		} else {
			num_mixed = add(
				s.mix_buffer[s.mix_buffer_offset:],
				data.samples[ps.offset:],
				AUDIO_MIX_CHUNK_SIZE,
				volume_start,
				volume_end,
				pan_stereo_start,
				pan_stereo_end,
			)
			
			ps.offset += num_mixed
			ps.offset_fraction = 0
		}

		// We didn't mix all the samples! This means that we reached the end of the sound.
		if num_mixed < AUDIO_MIX_CHUNK_SIZE {
			if ps.loop {
				ps.offset = 0
				ps.offset_fraction = 0

				// The sound looped. Make sure to mix in the remaining samples from the start of the
				// sound!
				overflow := AUDIO_MIX_CHUNK_SIZE - num_mixed

				if interpolate {
					samples_per_mixer_sample := (pitch*f32(data.sample_rate))/f32(AUDIO_MIX_SAMPLE_RATE)

					num_mixed = add_interpolate(
						s.mix_buffer[s.mix_buffer_offset + num_mixed:],
						data.samples[ps.offset:],
						ps.offset_fraction,
						overflow,
						samples_per_mixer_sample,
						volume_start,
						volume_end,
						pan_stereo_start,
						pan_stereo_end,
					)

					num_mixed_f32 := f32(num_mixed) * samples_per_mixer_sample
					fraction_advance := ps.offset_fraction + num_mixed_f32
					ps.offset += int(fraction_advance)
					ps.offset_fraction = linalg.fract(fraction_advance)
				} else {
					num_mixed = add(
						s.mix_buffer[s.mix_buffer_offset + num_mixed:],
						data.samples[ps.offset:],
						overflow,
						volume_start,
						volume_end,
						pan_stereo_start,
						pan_stereo_end,
					)

					ps.offset += num_mixed
					ps.offset_fraction = 0
				}
			} else {
				unordered_remove(&s.playing_sounds, idx)
				idx -= 1
			}
		}
	}

	ab.feed(out)
	s.mix_buffer_offset += AUDIO_MIX_CHUNK_SIZE
}
```