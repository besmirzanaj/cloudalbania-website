---
title: 'Fix Pulse Audio issues on USB bluetooth headsets'
date: "2022-04-27"
description: "fix pulse audio clicking sound in Ubuntu Linux"
draft: false
tags: 
  - pulseaudio
  - jabra
  - ubuntu
  - linux
---

- [Introduction](#introduction)
- [The problem](#the-problem)
- [The fix](#the-fix)

## Introduction

Using Ubuntu as my daily driver I have to also join work calls and one of the devices I use is a bluetooth headset with a USB dongle.

The dongle is an advanced bluetooth receiver paired to this headset and provides advanced audio features and encoding. 

## The problem

When using this headset with my laptop's built-in bluetooth receiver has limited capabilities in terms of high audio quality usage: if I use high fidelity audio, I cannot use the microphone :(.

To have high quality audio on this laptop seems I have to use the provided USB dongle from the vendor (Jabra). There is one caveat though. The sample rate of the USB device was different from that of the system. The Headset's sample rate was by default 48000 Hz and the system one was 44100 Hz. This caused clicks during phone calls and one could barely understand the other party.

default config in PulseAudio:
```bash
$ grep sample-rate /etc/pulse/daemon.conf
; default-sample-rate = 44100
; alternate-sample-rate = 48000
```

reading from the running daemon:

```bash
$ pulseaudio --dump-conf  | grep sample-rate
default-sample-rate = 44100
alternate-sample-rate = 48000
```

The kernel was complaining about this as the module was trying to use 48000 Hz but couldn't:

```console
[Wed Apr 17 10:15:19 2022] usb 1-1: Product: Jabra Link 370
[Wed Apr 17 10:15:19 2022] usb 1-1: SerialNumber: <SERIAL_NUMBER_HERE>
[Wed Apr 17 10:15:19 2022] usb 1-1: 1:1: cannot set freq 48000 to ep 0x3
[Wed Apr 17 10:15:19 2022] input: Jabra Link 370 as /devices/pci0000:00/0000:00:14.0/usb1/1-1/1-1:1.3/0003:0B0E:245D.0017/input/input64
[Wed Apr 17 10:15:19 2022] jabra 0003:0B0E:245D.0017: input,hiddev0,hidraw1: USB HID v1.11 Device [Jabra Link 370] on usb-0000:00:14.0-1/input3
```

Notice the ```usb 1-1: 1:1: cannot set freq 48000 to ep 0x3``` error message.

## The fix

I started troubleshooting by following [this article](https://jfreeman.dev/blog/2021/07/13/how-i-debugged-my-audioengine-hd3-speakers-in-linux/) and immediately it seemed obvious that the device was fighting with PulseAudio's settings. To fix that I updated the sample rates in ``/etc/pulse/daemon.conf`` to be the same at 48000 Hz so that there could be no more conflicts and then restarted PulseAudio with ```pulseaudio -k```. Now the running setting shows as below and all is working as expected.

```bash
$ pulseaudio --dump-conf  | grep sample-rate
default-sample-rate = 48000
alternate-sample-rate = 48000
```
