---
title: 'Aukey EP-B4 Bluetooth headphones: How to disable "Low battery, please charge" messages'
date: 2018-11-19T18:44:12+02:00
aliases: ["/posts/aukey-ep-b4-low-battery"]
---

A couple of years ago I bought Aukey's EP-B4 Bluetooth headphones so I could listen to some music in the gym without worrying about a wire. Instead I now worry about one more battery and an increasingly infuriating robotic voice.

<!--more-->

![Aukey EP-B4](/images/aukey-epb4.jpg)

These headphones are great and cheap. Good sound, fast connection, good enough battery life. BUT! When the battery is getting low, a robotic voice reports it over the music you are listening to every 5 seconds: _"Low battery, please charge"_. YES, EVERY FIVE SECONDS. I don't know who decided that was a good idea. I'd rather hear about it only once, and then keep listening to my music peacefully until the headset powers off. But no, that robotic woman will insist again and again, making them functionally unusable.

I then proceeded to search online for a solution. _"Maybe the firmware is available and I can somehow decompile it and modify that hardcoded `int`  to something a little less insane"_, I thought. But nope, there is no firmware online.

The only choice you have is to disable the voice notifications. A little _beep, beep_ will keep repeating every five seconds anyway, but the music won't stop abruptly. The steps are listed in the printed manual, but not in the [online version](https://images.aukey.com/en/downloads/EP-B4%20User%20Manul.pdf).

1. Headphones must be in pairing mode. This is, it cannot be connected to any other Bluetooth emitter.
2. Press `Volume Down` for three seconds.
3. The red LED will flash three times. Bingo!

This will switch off _all_ voice notifications, so the synthetic voice will not read the incoming calls out loud and so on. Totally worth it, IMO.