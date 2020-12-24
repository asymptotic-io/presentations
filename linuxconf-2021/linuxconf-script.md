# Introduction

Hello everyone. I am Sanchayan. I work at asymptotic.io. We are primarily an
open source consulting firm based out of Toronto and Bangalore. Our main focus
is on low level systems software with our primary work revolving around
GStreamer and PulseAudio or audio in general. Today I will talk about some of
the work I have been doing over the last 4 months as part of my work @
asymptotic to improve the state of bluetooth support in PulseAudio.

# What is PulseAudio

PulseAudio had it's first release in 2004 appearing for users in Fedora Linux
with version 8. While Advanced Linux Sound Architecture (ALSA) exists, which is
a software framework and part of the Linux kernel providing an API for sound
card device drivers, configuring ALSA can be tricky. Has limitations for
playing audio from multiple sources concurrently and lacks native Bluetooth
support. For example, Raspberry Pi OS recently switched to PulseAudio.

PulseAudio works on top of ALSA and primarily acts as a background process,
taking input from one or more sources and then redirecting these sources to one
or more sinks. So PA does the job of mixing. Without this only one program
would have been able to playback sound at a time.

Some of the helpful features PA provides are:

	- Per application volume controls
	- A plugin based architecture allowing support for loadable modules
	- Support for multiple sources and sinks
	- Ability to discover other computers using PA on local network and
	play sound through their speakers directly
	- Ability to combine multiple sound cards and synchronize multiple
	playback streams
	- Bluetooth support. Also provides ability to playback sound from
	computer's speakers by acting as audio for the remote bluetooth
	device.

There is also a frontend interface like pavucontrol allowing users to do some
of the configuration from the comfort of a GUI.

# Bluetooth Profiles

The bluetooth wireless technology standard has become pretty ubiquitous these
days. Devices have already started to come out with the latest Bluetooth 5.0
specification. Wanna buy a new cell phone? Make sure it has Bluetooth 5.0!

When talking about Bluetooth stack, one of the first thing that concerns us are
the various Bluetooth profiles. So what exactly are profiles? Profiles specify
behaviours that any bluetooth enabled device must adhere to or comply with, to
communicate with other bluetooth enabled devices.

The most commonly used profiles are
	* Advanced Audio Distribution Profile (A2DP)
	* Audio/Video Remote Control Profile (AVRCP)
	* Hands-Free Profile (HFP)
	* Headset Profile (HSP)

A2DP profile is where we have one device streaming audio to the other device.
This is what concerns most of us as we listen to music using headphones. AVRCP
allows a single remote control device to control all of the audio video
equipment in the immediate vicinity which a user might have access to. It is
more commonly used in car infotainment systems to control streaming of audio
over Bluetooth.

HSP is another commonly used profile. It allows the use of Bluetooth headsets
along with mobile phones. Provides abilities like to ring, answer a call, hang
up or adjust the volume.

HFP concerns itself with allowing hands free kits to communicate with mobile
phones like in car. In comparison to HSP, it adds extra features for use with a
mobile phone like voice dialing, call waiting or redialing the last number.

For this talk, henceforth, our focus will primarily be on the A2DP profile.

# Bluetooth Codecs

Now let's move on to codecs. So what is a codec? A codec is nothing but
hardware or software which encodes or decodes digital data. Now when it comes
to A2DP, it specifies one mandated codec and some optional codecs.

Low-complexity sub-band codec or more commonly known as just SBC, is the
mandated codec. It is what every Bluetooth device which supports A2DP must
implement at a minimum.

There are other codecs available. Qualcomm has it's Audio Processing Technology
aptX codec. Sony has LDAC. There is also Advanced Audio Coding (AAC), which has
been standardised by ISO and IEC as part of MPEG-2 and MPEG-4 specifications.
All the last three require licensing and license fees.

All devices support SBC but LDAC is generally available with Sony and some
other DACs while aptX is generally only available with devices which use
Qualcomm chipsets. Apple devices primarily do AAC.

# So which is better?

So we basically live in this XKCD comic world where we actually have 14 codecs.
aptX has some more variants. There seem to be some other codecs from Samsung
and Huawei though I haven't seen any of those myself. There is FastStream which
is some SBC variant and can be used for better audio quality with HSP. However,
it is difficult to find headsets supporting faststream.

(Talk about some points from https://habr.com/en/post/456182/)

One of the downsides of A2DP specification is, that while it mandates the SBC
codec, the quality of output from SBC can vary widely depending on how some of
the codec parameters are configured. A2DP standard does not define fixed profiles
but only gives recommendations. It can be considered to be the best and
the worst codec at the same time. 

One such SBC codec parameter is what is known as bitpool value. Even if a
manufacturer sets the maximum allowed value for high quality, the OS bluetooth
stack might not allow using that. As a result, some manufacturers might
conservatively set the low maximum value for some devices.

With aptX there is nothing that the user or a manufacturer can control.
Qualcomm distributes the reference library for the codec.

AAC theoretically should produce an output which is very close to the original.
However, it has been observed that Apple seems to have the best encoder
implementation available with it's macOS and iOS. Second best is the Fraunhofer
FDK. AAC is a processing heavy codec and on Android phones with Energy Aware
Scheduling, where energy efficiency will be prioritized over performance, AAC
encoding will be done with a lower bitrate and quality. This results in a poor
performance on Android phones.

LDAC is a new codec promoted by Sony often advertised as a codec meant for
audiophiles. It supports Adaptive Bit Rate viz. changing the bit rate depending
on transmission conditions. Decoder is not freely available and the reference
encoder implementation is the one included with Android. As per the tests done
by folks at soundguys, it can provide slightly better results compared to
aptX-HD.

# Current State of PulseAudio

Now, let's talk about the current state of things in PulseAudio. Currently
upstream only supports SBC codec. HSP and HFP are supported however the sound
quality has been a pain point for many users. There has been an out of
community effort providing support for other codecs, but, the way it implements
things does not make it suitable for merging upstream.

# Challenges

One of the first things users are gonna expect along with the support for
additional codecs is the ability to switch between them. Till as recently as
last December,  this was not possible. The support required to implement
switching between different codecs only got merged in December.

When it comes to codecs, always there are various license considerations to
think of.

Now, what if we want to support multiple encoder or decoder implementations.
For example, in the case of AAC we have competing implementations between
FFMPEG and Fraunhofer FDK.

Just like quite a few open source projects, PulseAudio is currently maintained
by three volunteer developers on their own free time (probably amounting to
less than one full time developer), which is not really enough, given the
project size and scope.

# GStreamer

GStreamer is a pipeline-based multimedia framework. The goals of GStreamer is
to separate the application from the underlying complexities of dealing with
encoding, decoding or streaming of media.

This is best explained with an example. So, let's look at one.

Consider the example for playing audio over Bluetooth while using LDAC
encoding.

gst-launch-1.0 -v audiotestsrc !
audio/x-raw,rate=44100,channels=2,format=S32LE ! ldacenc eqmid=2 ! a2dpsink
transport=/org/bluez/hci0/dev_4C_BC_98_80_01_9B/sep10/fd0

gst-launch is a tool which builds and runs GStreamer pipelines. The pipeline
description is a list of elements separated by exclamation marks. Properties
are appended to the elements in form of **property=value**.

# Progress so far

For the last 4 months we have been working to improve the whole Bluetooth
story.

We first wanted to support LDAC. This took about 3-4 weeks of September. LDAC
support in GStreamer got merged upstream by end of October. For supporting AAC
with bluetooth, we needed to add support for an AAC variant which wasn't
present in GStreamer. This required another 3-4 weeks of work, adding support
to the required plugins. 

We initially intended to provide support for aptX via FFMPEG. However, the
aptX-HD encoder there introduces a delay and while decoding, we observed delays
and stuttering for both encoding and decoding. There has been a libopenaptx
library available which seems to solve both these issues. We were grateful that
a gstreamer implementation based on libopenaptx had just got merged.

We then started working for testing this along with PulseAudio. With the
required support in PulseAudio to implement codec switching finally merged in
December, we could implement the codec switching bits. The patch series as of
this recording has been posted and is currently under review.

# Conclusion

So what's next? First of all we need to implement whatever feedback we receive
on our merge request. While we have done testing on our end, users and other
maintainers will test further and probably report bugs or missing pieces. We
will be dependent on GStreamer 1.20 landing in various distributions which will
include support for LDAC, aptX via libopenaptX library and AAC. Finally, the
PulseAudio release which includes all this being packaged for various
distributions.
