---
layout: default
title: Making music with via MIDI with Analogue Pocket + mGB
tags: retro gaming audio chiptune mgb
author: Marc_Alx
---

_published first on [reddit.com](https://www.reddit.com/r/AnaloguePocket/comments/18eks6y/tutorial_analogue_pocket_mgb_midi/)_

# Making music with via MIDI with Analogue Pocket + mGB

Here's a small tutorial on how to make music on Analogue Pocket using a midi controller.

*disclaimer: I'm not pro in this domain, but it's was hard for me to find a clear tutorial.*

## The setup

- Analogue Pocket
- [openFPGA-GB](https://github.com/spiritualized1997/openFPGA-GB-GBC)
- [mGB 1.3.3](https://github.com/trash80/mGB)
- [Arturia Beatstep Pro](https://www.arturia.com/products/hybrid-synths/beatstep-pro/overview) (should work with other midi controller)
- [Arduinoboy MIDIGBX](https://www.etsy.com/fr/listing/1582128940/adaptateur-arduinoboy-gameboy-midiin)
- Gameboy link cable (one that fits Analogue GB port)
- MIDI cable

## Connections

- Analogue Pocket connected via link cable to Arduinoboy (don't mess with cable orientation on Arduinoboy)
- Arduinoboy connected (midi IN) to MIDI controller (midi OUT port) via MIDI cable

## Usage

- Launch mGB
- Start Arduinoboy
- Set Arduinoboy to mGB mode, see [doc](https://github.com/trash80/Arduinoboy)
- Play music by sending note on midi channels 1, 2, 3, 4. Each channel control one GB Channel.

If everything is ok you should see Arduinoboy blinking on each note sent, and (obviously) hear notes on your Analogue Pocket.

## Q&A

**Why all this setup, are Analogue Cable sufficient (Nanoloop to Pocket MIDI IN)?**

Analogue cable only work with Analogue pocket builtin Nanoloop. Moreover this cable only transfer midi sync, not notes.

**So I need an Arduinoboy?**

Yes, it converts MIDI signals to the format expected by mGB and other softwares.

**How do I record?**

Maybe by recording through Analogue jack port.

**Why Arturia beatstep pro?**

It's not a mandatory requirement, any MIDI controller should work. This one is cool as it sends notes on multiple channels simultaneously by recording pattern.

**Everything is plugged, no sound...**

- Is sound enable (not muted) on Analogue pocket?
- Are you using the right MIDI channel (1 to 4 only).
- Did you respect MIDI IN/OUT recommendations?
- On Arduinoboy is the GB cable in the right orientation? Try flipping upside down 
- Is Arduinoboy plugged to sector? (some led should light)
- Is your midi controller plugged to sector?
- Check that the channel your are targeting is not muted in mGB. Either on L, R, LR configuration.

**mGB alone produce no sound, is it normal?**

Yes, mGB alone without a MIDI controller is just a software to configure the 4 audio channels of the Game Boy. It only produces sounds when you send MIDI notes through the described setup.

**Can it be simple?**

If Analogue implements Arduinoboy behaviors inside Analogue OS, along with notes support for its MIDI cable it may be simple. This way Arduinoboy would be optional.
