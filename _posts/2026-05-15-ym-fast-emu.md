---
layout: post
title: Playing ATARI music on Amiga for free!
subtitle: How to emulate an Atari YM2149 audio chip on Amiga at litteraly 0 CPU cost, by Arnaud Carré
cover-img: /assets/img/ymemu/fastym_title.jpg
thumbnail-img: /assets/img/ymemu/fastym_thumb.jpg
comments: true
tags: [atari, amiga, audio, emulation]
---
## Audience

This post is for anyone who loves the technical and historical aspects of chiptune music, as well as enthusiasts of the Amiga PAULA and Atari YM2149 audio chips.

## The Context

In my [Cycle-Op demo](https://www.pouet.net/prod.php?which=94129), I showcased a classic [sin-dots effect](https://youtu.be/Mof0Cv4A98M?t=231) that rendered **6405** dots at 50 FPS on the Amiga 500. Two years later, Amiga legend Hannibal released the long-awaited [3D Demo 3](https://www.pouet.net/prod.php?which=103979), packed with a lot of impressive effects. Among them was his own sin-dots record, beating mine with **6682** dots.

To top it all off, and in the classic demo scene tradition of playful roasting, Hannibal left me this message:

> "Hi Leonard, you optimized your dots well for an Atari programmer. But there were hundreds of dots left if you optimize like an Amiga expert."

That was too good to ignore. I had to respond by finding a way to beat his new sin-dots record and, as a bonus, come up with a clever twist to answer this "Atari programmer" jab :)

## The Goal

As a nod to Hannibal's remark, I immediately had the idea of playing Atari music on the Amiga during my future dot record attempt. To do that, I needed to emulate the YM2149 sound chip. I had already written an Atari music emulator for the Amiga back in time in my [AmigAtari demo](https://www.pouet.net/prod.php?which=85276).

However, accurately reproducing modern Atari music effects, such as SID voices, Sync Buzzer, and Digidrums, requires emulating not only the YM2149 but also the Atari's hardware timers. This kind of emulation is extremely CPU intensive, consuming around 50% of the frame time in my 2020 AmigAtari demo. That makes it impossible to break a sin-dots record at the same time.

So I needed a much simpler solution: a way to play Atari music on the Amiga without using any CPU at all.

My first rough idea was straightforward: Why not use the Amiga PAULA chip itself to emulate the YM2149, leaving the poor Motorola 68000 entirely dedicated to drawing sin dots?



## YM2149 and PAULA

On paper, these two chips are very different.

The **YM2149** is a slightly modified version of the AY-3-8910, rebranded by Yamaha. It is a relatively simple sound chip. It has three voices capable only of generating square waves. A pseudo-random noise generator and a basic hardware volume envelope, intended as a cheap [ADSR envelope](https://en.wikipedia.org/wiki/Envelope_(music)#ADSR) substitute

**PAULA**, the Amiga's audio chip, takes a completely different approach.
It is a PCM sample playback chip with no built-in support for square waves, noise generation, or envelopes. PAULA can play four independent signed 8bit PCM samples directly from main memory, each with its own playback rate and volume.

For a chip introduced in 1985, this design was remarkably advanced.


## First Idea and Prototype

Most Atari music relies heavily on square waves, which made the initial concept seem simple. I stored a single cycle of a square wave in memory as 8bit samples and played it in a loop using one PAULA voice.

To avoid porting a full Atari music driver to the Amiga, I used a technique similar to the one I developed for my ultra fast [Amiga MOD player, LSP](https://github.com/arnaud-carre/LSPlayer) : pre-compute all PAULA state per frame in a dedicated data stream.

The plan was:

- Write a PC tool that reads an Atari .sndh music file and simulates it frame by frame
- Extract the period and volume values of the three YM voices at 50 Hz
- Convert these values into PAULA-compatible period and volume values
- Store the result in a dedicated data file

On the Amiga side, playback became extremely lightweight: each frame, the program simply reads a few values and updates the period and volume of three PAULA channels, each looping a tiny square wave sample.


## First boring result

What kind of Atari music can you play using nothing but square waves? Let's try Buggy Boy, a very old ATARI game.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <video controls preload="metadata"
         style="position: absolute; top:0; left:0; width:100%; height:100%;">
    <source src="{{ '/assets/img/ymemu/buggyboy.mp4' | relative_url }}" type="video/mp4">
  </video>
</div>

Not very exciting, is it? No offense to the original Atari port, but pure square waves sound rather bland and certainly not epic enough to accompany a future Amiga sin-dots world record.

I needed something better.


## YM Envelope Trickery

By the late 1980s, several talented demoscene musicians had discovered ways to coax much richer sounds from the YM2149. One of the most influential was Jochen Hippel, better known as MadMax of TEX.

The YM envelope hardware, originally designed as a simple ADSR substitute, was rarely used as intended. The main reason is that the chip provides only a single envelope generator shared across all three voices. This means all channels would have to use exactly the same envelope shape.

But inventive musicians discovered a great work around. Rather than using the envelope to modulate the volume of a square wave, they used the envelope itself as the sound source. This produced the sweeping, buzzing timbres heard in many classic Atari soundtracks, such as the legendary Thalion Software intros:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <video controls preload="metadata"
         style="position: absolute; top:0; left:0; width:100%; height:100%;">
    <source src="{{ '/assets/img/ymemu/thalion.mp4' | relative_url }}" type="video/mp4">
  </video>
</div>


**How the hell my poor YM could output such a bad ass sound?**

For anyone who grew up listening to simple tunes like Buggy Boy, hearing this for the first time was a revelation. There is no official name for this distinctive sweeping effect, but for the sake of this article, let's call it the **MadMax Buzzer**.


## How It Works

The YM2149 provides several predefined volume envelope shapes, as shown in the diagram below.

<div align="center">
  <img src="/assets/img/ymemu/ymr13.jpg">
</div>

Some of these shapes are triangle waves, some are sawtooth waves.

Now imagine the following setup: The envelope is enabled for the voice and configured to use a triangle shape. Envelope frequency is set high enough to fall within the audible range. ( This makes no sense as a traditional ADSR control, but now the envelope itself becomes the sound source )

The result is an audible waveform similar to a triangle. That already sounds great, but it still does not quite match the aggressive and sweeping sound heard in the MadMax example. What is the missing ingredient? Instead of just playing the envelope, you also enable the square wave on the voice, using a slightly detuned frequency.

When square wave signal is 0, output is 0. When square is 1, output is the current envelope triangle shape! The result is a complex and moving wave, typical of MadMax buzzer! Let's hand draw this

<div align="center">
  <img src="/assets/img/ymemu/ymenv.gif">
</div>


As both frequencies are slightly detuned, it produces a moving, sweeping typical MadMax sound! Now you can see how this works by looking at **voice B** in the classic "Leavin Teramis" MadMax track. Notice how the triangle envelope shape evolves over time due to the detuned square wave.

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <video controls preload="metadata"
         style="position: absolute; top:0; left:0; width:100%; height:100%;">
    <source src="{{ '/assets/img/ymemu/leavinteramis9.mp4' | relative_url }}" type="video/mp4">
  </video>
</div>

_Note: The triangle waveform appears quite rounded because the YM2149 enveloppe is logarithmic instead of linear (and also only contains 32 discrete values)_
{: style="background-color: #fff9c4; padding: 10px; border-radius: 5px; display: block; font-size: 0.9em;"}

## How to emulate this envelope tricks with PAULA?

PAULA is a surprising chip. A long time ago I remembered reading something about the ability to modulate period or volume. Looking at PAULA hardware manual you can see there is a mode called "attached" voice. In this mode, one PAULA voice could be used to modulate the period or volume of another voice.

In a way, this feature is similar to the YM2149 ADSR envelope. Not technically, but because both features are mostly ignored by Atari and Amiga programmers! :)

To my knowledge, no Amiga game or demo actually uses it. (Probably because it is not very practical as an ADSR replacement: it consumes one precious PAULA voice per modulated voice.) Instead, Amiga trackers drive the voice volume at 50 Hz using the CPU. It is not as accurate as per-sample volume modulation, but it is more than good enough for games and demos.

The YM2149 has 3 voices. PAULA has 4. So why not use 3 of them to emulate YM square waves, and use the 4th PAULA voice as a volume modulator? That sounds like a solid plan! An ATARI programmer might be among the first to make use of this underrated PAULA feature :)

Let us take a look at the modulator data format described in the PAULA manual.

<div align="center">
  <img src="/assets/img/ymemu/pauladoc2.jpg">
</div>

In normal mode, each 16bit word represents two 8bit signed PCM samples (blue circle). In the attached "volume modulation" mode, each 16bit word represents a single PAULA volume value (green circle).

As a result, the modulation data runs at half the rate of the audio sample playback (one volume value for every two 8bit samples). More on that later.

In Amiga memory, we now have an 8bit square wave:

```asm
AmigaSquareWave:
	dc.b	$7f,$7f,$7f,$7f,$7f,$7f,$7f,$7f,$00,$00,$00,$00,$00,$00,$00,$00
```

_Note: I'm not using full range square wav (-128,+127]) but (0..127) so half of square cycle is 0 instead of -128, to properly generate a 0 output when modulated by envelope_
{: style="background-color: #fff9c4; padding: 10px; border-radius: 5px; display: block; font-size: 0.9em;"}

And also a 16bits triangle wave, representing volume values. (remember volume modulation data is 16bits, despite it contains only PAULA volume values in the range of [0..64])

```asm
AmigaVolumeModulatorTriangleWave:
	dc.w	0,0,0,0,0,0,0,0,1,1,1,1,2,2,3,3
	dc.w	4,5,6,7,9,11,13,15,19,22,26,31,38,45,53,64
	dc.w	64,53,45,38,31,26,22,19,15,13,11,9,7,6,5,4
	dc.w	3,3,2,2,1,1,1,1,0,0,0,0,0,0,0,0
```
_Note: It's not linear triangular shape because of the logarithmic nature of YM2149 volumes_
{: style="background-color: #fff9c4; padding: 10px; border-radius: 5px; display: block; font-size: 0.9em;"}


When my PC offline tool detects both YM Envelope and Square on a voice, Amiga replayer will enable this PAULA attached mode, and modulate the volume of voice 1 using our 16bits triangle shape wave from voice 0. Let's do a test by isolating MadMax buzzer in **Leavin Teramis, song 9**.

First is the audio reference, generated on PC using my [Atari Audio emulation library](https://github.com/arnaud-carre/sndh-player/tree/main/AtariAudio).

<audio controls>
  <source src="{{ '/assets/img/ymemu/leavin9_pc_ref.mp3' | relative_url }}" type="audio/mpeg">
  Your browser does not support the audio element.
</audio>

And then the emulation result, outputed form WinUAE Amiga emulator, using the weird PAULA "attached voice" feature:

<audio controls>
  <source src="{{ '/assets/img/ymemu/leavin9_amiga_bad.mp3' | relative_url }}" type="audio/mpeg">
  Your browser does not support the audio element.
</audio>


Damn, it sounds pretty bad. It doesn't sound like the ref at all... Why is that?


## Audio Debugging

Let's have a look at these two outputs using Audacity.

Here is the reference from AtariAudio PC emulation library:

<div align="center">
  <img src="{{ '/assets/img/ymemu/audacity_ref.png' | relative_url }}">
</div>

And now the Amiga WinUAE ouput using the "attached voice" modulation trick

<div align="center">
  <img src="{{ '/assets/img/ymemu/audacity_amiga_bad.png' | relative_url }}">
</div>

We can clearly spot the issue here. The modulator envelope looks quite coarse: the expected smooth triangle shape has very low resolution, with large, noticeable volume steps.

This is easy to explain. Remember that in attached voice mode, we only get one volume value for every two output samples. When we sample our triangle wave at the required music rate, many intermediate values are simply skipped. The result is this chunky waveform.

I scratched my head for a while, trying to tweak the wave resolution, the modulator pitch, and other parameters. Nothing really worked. No matter what I tried, the low-rate nature of the modulator data always produced the same kind of coarse result.

I was about to give up and just use a classic Amiga MOD for my future sin-dots record...

and then...

## The "Eureka" moment!

One day, lying in bed at night, the idea came to me! (I have noticed that most "eureka" moments happen just as I am about to fall asleep :) )

Remember how Atari musicians pioneered creative techniques by using the YM2149 envelope in ways never intended by the original chip designers? Why not do the same with PAULA?

The attached voice low-frequency data is supposed to be the envelope shape, modulating an 8bit square signal. But what if we simply reverse the roles? Let us store the triangle envelope as the 8bit sample, and use a square "volume" wave as the modulator. The result should be the same, but a square wave does **not** require the high resolution that a triangle wave does! 

Just few lines of code to change. Now the square wave is stored in the 16bit modulator format (high is 64, low is 0).

```asm
AmigaSquareWaveAsModulator:
	dc.w	64,64,64,64,64,64,64,64,0,0,0,0,0,0,0,0
```

And triangle envelope shape will be played by standard 8bits PCM, with more resolution (0..+127)

```asm
AmigaVolumeModulatorTriangleWaveAs8bitsPCM
	dc.b	$00,$00,$00,$00,$01,$01,$01,$01,$02,$02,$03,$03,$04,$05,$06,$07
	dc.b	$09,$0b,$0d,$0f,$12,$16,$1a,$1f,$25,$2c,$35,$3f,$4b,$59,$6a,$7f
	dc.b	$7f,$6a,$59,$4b,$3f,$35,$2c,$25,$1f,$1a,$16,$12,$0f,$0d,$0b,$09
	dc.b	$07,$06,$05,$04,$03,$03,$02,$02,$01,$01,$01,$01,$00,$00,$00,$00
```

Let's look at the WinUAE output!

<div align="center">
  <img src="{{ '/assets/img/ymemu/audacity_amiga_fixed.png' | relative_url }}">
</div>

And listen to the new AMIGA output!

<audio controls>
  <source src="{{ '/assets/img/ymemu/leavin9_amiga_fixed.mp3' | relative_url }}" type="audio/mpeg">
  Your browser does not support the audio element.
</audio>

It works!

## Finally the 0 CPU

So far, so good: we can play nice Atari music with that MadMax buzzer-like sound. Our player is also very fast (less than a raster line), since it only setup a few PAULA registers per frame. But did we not promise **"0% CPU load"** in the title?

Amiga is a great machine, packed with custom chips. One of them is the COPPER. The COPPER is a very simple yet extremely powerful coprocessor that runs fully in parallel with the 68000 CPU. It executes its own instructions from a "COPPER list" and supports only two types of operations:

- write an immediate value into any Amiga custom chip register (including blitter, colors, screen, PAULA)
- wait for a specific screen position

So why not pregenerate tons of tiny COPPER lists, one per music frame? Each will contain all the required "write" commands to the PAULA registers. This would completely offload the 68000. Since the COPPER can also update the pointer to the next COPPER list, we can build a fully automatic, self-chaining sequence that plays Atari music on its own.

In the end, the YM2149 emulator truly uses **0% of the CPU**! Not even a **single 68000 instruction**! :)

## Final result

Here is the resulting new sin-dots record! We managed to display **7210** dots instead of **6682**, while playing Atari music at the same time! Not bad, right? :) (I may write another blog post explaining how we achieved this 7210-dot result.)

Also, enjoy the great Atari music by BigAlec, showcasing the MadMax buzzer-like effect at its best!

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <iframe src="https://www.youtube.com/embed/2Al9SrsB47I"
          style="position: absolute; top:0; left:0; width:100%; height:100%;"
          frameborder="0"
          allowfullscreen>
  </iframe>
</div>

You can also [download the demo here](https://www.pouet.net/prod.php?which=104190).

Enjoy!






