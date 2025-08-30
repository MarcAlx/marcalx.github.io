---
layout: default
title: Audio emulation 101 - Game Boy example
tags: emulation audio game-boy sound gb
author: Marc_Alx
---

_published first on [reddit.com](https://www.reddit.com/r/EmuDev/comments/1mx2k5e/audio_emulation_101_game_boy_example/)_

# Audio emulation 101: Game Boy example

*Disclaimer: this post is not a full breakdown, it's just a compilation of information I wish I had before starting. I will focus on Game Boy as it's the only system I emulate.*

After achieving audio emulation on my [Game Boy emulator](https://github.com/MarcAlx/GBKit) I think it's a good time to summarize some key things I've understood in the process.

First, audio (as for the Game Boy) is way less documented and straightforward than CPU or Graphics (PPU for Game Boy). As always reading the doc and reference is key to understand what's happening.

**1. Video vs. Audio**

Usually when making an emulator everyone understands what video memory is and how an image should be produced from that, but for audio it's usually obscure.

**Important notion:** Back-end VS Front-end, back-end usually refers to your emulation logic where front-end refers to how your emulation is interacted with (both input and output). Do not merge these two notions, separate them in your code, *i.e do not put SDL code in your APU; make two modules*.

Let's make a comparison between audio and video, from a front-end perspective. 

* Video is a succession of frames (images) made of pixels rendered at a fixed frame rate (usually 60 frames per second). Each pixel is made of three components (RGB) and your front-end tells you how to arrange them to make an image.
* Audio is a succession of samples (a fixed size buffer) made of float numbers (two, one for each stereo channel: left and right) played at a fixed rate/frequency (usually 44.1 kHz or 48 kHz). These floats are not as granular as a music note. This is the succession of these floats that represents a wave that all along produces sound. This wave usually has values that range from -1.0 to 1.0, where a succession of 0.0 produces no sound. How you should expose your samples, how many samples you should provide, how long a buffer would be heard, and how you handle stereo (interleave of L and R values) depends on your front end (SDL...).

*Summary*

|Concept|Video|Audio|
|:-|:-|:-|
|Unit|Frame|Buffer|
|Made of|Pixels|Samples|
|Components|RGB /RGBA|L, R Channels|
|Rate naming|Frame rate|Sample rate|
|Typical rate|60-30 FPS|44.1-48 kHz|

**A note on audio latency:** usually emulators are synced over video, so *"how is audio synced?"* Basically you should continuously produce audio buffers for your front-end to play. If your buffers are too large they take time to fill thus latency, too small they may be consumed too fast leading to pauses (little *"poc"* heard). Try different values, or respect what your front-end expects to achieve good sync.

**2. Game Boy example**

Here's a quote from pandoc: *"The PPU is a bunch of state machines, and the APU is a bunch of counters."*. Why audio (in the case of Game Boy) is referred to as counters? Mainly because through the game various audio parameters are adjusted after a certain number of ticks (you should count). Game developer controls these timings through registers along with other parameters to produce sound.

**2.1 Channels**

Game Boy is composed of 4 channels, each of these channels may be seen as an instrument that makes a particular sound. Each channel produces a wave at its own rate/frequency (like we discussed  before). The value on the wave at one point is called amplitude.

**The 4 channels are:** 2 Square channels, 1 wave channel, 1 noise channel.

**Square** channels produce predefined (by hardware) waveform made of up (1) and down (0). Resulting sample is made of 0 (when down) or value (when up), where value corresponds to the volume of the channel.

One of these square channels is called sweep; this sweep eases channel frequency adjustment to make cool audio effects. This allows for audio pitch/tone control.

Third channel is **Wave**; it produces a user-defined wave (by wave RAM). Resulting sample is made of the currently played portion of wave RAM. There's no volume on this channel, the volume is controlled by another parameter that shifts the value to adjust volume.

Fourth channel is **Noise**, a random succession of 0 and 1 (produced via a certain formula you should emulate: LFSR). Resulting sample is 0 (when 0) or volume (when 1).

Each channel has a length parameter, which means "after a certain time stop playing".

Channel 1,2, and 4 have an envelope parameter to control volume dynamically. 3 has none as it has its own volume logic. These parameters (along with sweep) are controlled via a step sequencer: an internal counter that runs inside the APU and steps these parameters at a fixed rate.

*Summary*

* **Channel 1 (Square+Sweep) →** Square wave with pitch/tone control.
* **Channel 2 (Square) →** A simple square wave
* **Channel 3 (Wave) →** User defined wave
* **Channel 4 (Noise) →** Random noise

|Concept|Video|Audio|
|:-|:-|:-|
|Processing unit|PPU|APU|
|Components|Layer (BG, WIN, OBJ)|Channels (Sweep, Pulse, Wave Noise)|

**2.2 Sampling**

*"Ok each channel produces a wave, at any time I can observe this value to get the amplitude of the channel. But how do I produce samples?"* 

To produce one sample from these 4 channel values you should merge them by applying master volume and audio panning.

Audio panning tells for each channel if value should apply to left or right ear (if not it's 0 for that ear, thus silence). This is stereo sound and it allows some audio effects.

Master volume which applies a factor to L and R.

The merging process is a sum of each channel value, distributed on L or R according to pan, multiplied by master volume and divided to range between -1, 1.

**Important note:** each Game Boy channel amplitude ranges from 0x0 to 0xF, it's up to you to implement digital (uint) to analog (float) conversion (DAC) to make it range to -1, 1.

**3. How do I debug audio?**

That's the hardest part, isolate a channel or a channel component (Envelope, Length...). Compare with hardware, or reliable emulator such as Same Boy (which allows per channel isolation).

Try to produce an audio .wav file to view resulting wave. 

Don't spend too much time on test roms. They often rely on obscure behaviors and may lead to false-positive bug cause.

**Pro tip:** quickly add parameters to your emulator to mute some channels, this will ease debugging.

**4. What we didn't discuss here**

* How each channel is timed/ticked
* How the step sequencer works
* The high-pass filter that is applied and the end of sampling (optional for basic audio emulation). 

**Final notes**

Thanks for reading (and to /emudev community), it's just an intro have a look at my emulator ([back-end](https://github.com/MarcAlx/GBKit/tree/main/Sources/GBKit/Components/APU)/[front-end](https://github.com/MarcAlx/gb/blob/main/src/packages/GBUIKit/Sources/GBUIKit/Managers/AudioManager.swift)) to better understand how it works, even if not 100% accurate there's a lot of comments and links in the readme.

*PS: achieving perfect audio emulation is very hard, there's a lot of obscure behavior, tricks used by developers. But producing good audio (at least for Game Boy) is achievable in <1000 SLOC, you can do it!*