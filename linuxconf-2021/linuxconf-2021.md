---
title: 
- What's next for Bluetooth in PulseAudio?
author:
- Sanchayan Maity
institute:
- asymptotic.io
theme:
- default
classoption:
- aspectratio=169
---

# What is PulseAudio?

- Initial release on 17 July 2004
- First appeared for users in Fedora Linux with version 8
- Mediates access to audio resources on your system
- Features?
	* Audio mixing
	* Per application volume controls
	* Multiple sources and sinks
	* Combine multiple sound cards and synchronize multiple playback
	streams
	* Bluetooth support
	* Command line interface with scripting capabilities
	* Sound daemon with reconfiguration capabilities

# Bluetooth Profiles

- What is a profile?
- Some of the widely used bluetooth profiles
	* Advanced Audio Distribution Profile (A2DP)
	* Audio/Video Remote Control Profile (AVRCP)
	* Headset Profile (HSP)
	* Hands-Free Profile (HFP)

# Bluetooth Codecs

- Available codecs?
	* Low-complexity subband codec (SBC)
	* Audio Processing Technology (aptX, aptX-HD)
	* Advanced Audio Coding (AAC)
	* LDAC

# So which is better?

![*_XKCD - Standards_*](../images/standards.png){width=80%}

# Codec Latencies!

![*_Soundguys - Android Bluetooth Latency_*](Bluetooth-Codec-Latency-by-Smartphone.jpg){width=70%}

# Current State

- Upstream only supports SBC :(
- HSP/HFP support 
- Community efforts: out-of-tree pulseaudio-modules-bt, another implementation submitted upstream
- Messaging API

# Challenges

- Codec switching
- Patent encumbered licenses?
- How to support multiple encoders or decoders?
- Contributors? Maintainers?

# GStreamer

- What is GStreamer?
- Why?

# Example Pipeline

```bash
gst-launch-1.0 -v audiotestsrc !
audio/x-raw,rate=44100,channels=2,format=S32LE !
ldacenc eqmid=2 ! a2dpsink
transport=/org/bluez/hci0/dev_4C_BC_98_80_01_9B/sep10/fd0
```

![](ldac.svg "")

# Progress so far

- LDAC support upstreamed in GStreamer
- Support for Low-Overhead MPEG-4 Audio Transport Multiplex (LATM) AAC in GStreamer
- GStreamer wrapper around *libopenaptx*. Work done by Igor Kovalenko.
- Implementing codec switching
- Proof of concept with GStreamer tested
- Merge request opened in PulseAudio

# Questions?

- So what's next?
	* Support Adaptive Bit Rate (ABR) in LDAC
	* mSBC support for HFP profile
	* LC3 codec
	* Allowing users to specify the preference order for codecs
	* Support Opus as a vendor codec for PulseAudio <-> PulseAudio
