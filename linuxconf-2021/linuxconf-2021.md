---
title: 
- What's next for Bluetooth in PulseAudio?
author:
- Sanchayan Maity
institute:
- asymptotic.io
theme:
- Berlin
classoption:
- aspectratio=169
---

# What is PulseAudio?

- Initial release on 17 July 2004
- First appeared for users in Fedora Linux with version 8
- Features?
	* Audio mixing
	* Per application volume controls
	* Multiple sources and sinks
	* Combine multiple sound cards and synchronize multiple playback
	streams
	* Bluetooth support
	* Command line interface with scripting capabilities
	* Sound  daemon with reconfiguration capabilities

# Bluetooth Profiles

- What is a profile?
- Some of the widely used bluetooth profiles
	* Advanced Audio Distribution Profile (A2DP)
	* Audio/Video Remote Control Profile (AVRCP)
	* Hands-Free Profile (HFP)
	* Headset Profile (HSP)

# Bluetooth Codecs

- Available codecs?
	- Low-complexity subband codec (SBC)
	- Audio Processing Technology (aptX, aptX-HD)
	- Advanced Audio Coding (AAC)
	- LDAC

# So which is better?

![](../images/standards.png){width=80%}

# Current State

- Upstream only supports SBC :(
- HSP/HFP support
- Out of tree community effort with [pulseaudio-modules-bt](https://github.com/EHfive/pulseaudio-modules-bt)

# Challenges

- Codec switching
- Patent encumbered licenses?
- How to support multiple encoders or decoders?
- Contributors? Maintainers?

# GStreamer

* What is GStreamer?
* Why?

# Example Pipeline

```bash
gst-launch-1.0 -v audiotestsrc !
audio/x-raw,rate=44100,channels=2,format=S32LE !
ldacenc eqmid=2 ! a2dpsink
transport=/org/bluez/hci0/dev_4C_BC_98_80_01_9B/sep10/fd0
```

![](ldac.png "")

# Progress so far

- LDAC support upstreamed in GStreamer
- Support for Low-Overhead MPEG-4 Audio Transport Multiplex (LATM) AAC in GStreamer
- GStreamer wrapper around *libopenaptx*. Work done by Igor Kovalenko.
- Implementing codec switching
- Proof of concept with GStreamer tested
- Merge request opened in PulseAudio

# Questions?

#### Thank You!
