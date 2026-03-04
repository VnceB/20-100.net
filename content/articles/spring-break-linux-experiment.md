---
title: "My Son's Spring Break Linux Experiment (and the WiFi Rabbit Hole That Almost Killed It)"
date: 2026-03-04
tags: ["linux", "drivers", "claude-code", "open-source"]
---

My son wanted to try Linux over spring break. His theory: ditching Windows would get him better FPS out of his GPU. I knew it probably wouldn't make a difference, but he was excited, and I was happy he wanted to give it a shot.

We installed Fedora 43 on his desktop and the WiFi didn't work. The MT7902 chipset on his motherboard doesn't have a supported driver in Fedora 43's kernel. Freshly installed OS, no internet. He was already losing interest.

I fired up Claude Code in the terminal and described the problem. It found an experimental MT7902 driver built for the same kernel version that Fedora 43 ships. But it didn't compile. The firmware installer was pointing at the wrong files, and Secure Boot was rejecting the unsigned module.

So Claude Code patched the Makefile to handle MOK signing, fixed the firmware paths, and wrote a DKMS config so the driver would rebuild itself on kernel updates. I enjoy digging into this kind of stuff, but my son would have been long gone by that point.

WiFi came up. We were finally online.

We pushed the fix to GitHub so nobody else has to deal with this: [github.com/VnceB/mt7902_temp](https://github.com/VnceB/mt7902_temp). Figured it was a good chance to show my son how open source works. Someone else's code got us out of a jam, so we put our fix out there too.

Without Claude Code we just wouldn't have done it. I don't have hours to dig through kernel mailing lists and debug module builds, and a kid on spring break has zero patience for any of that. We got it working in one sitting and moved on. The FPS never improved, and his favorite game kept crashing mid-match during competitive play, which got him banned for a few hours. Without the FPS gain, he wasn't about to gamble his profile's reputation on more debugging sessions with dad. He's back on Windows.
