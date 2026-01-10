---
title: Building a Chromatic Tuner with Claude - Part 1
date: "2026-1-7"
---

## A Word of Encouragement
If you have not tried using a AI coding assistant to build something, the time is now. Honestly, you are probably ~12 months past due.

Perhaps you are one of the many thousands of recently unemployed software engineers, scrolling aimlessly through LinkedIn, practicing LeetCode and firing off resumes into the abyss. If that is you, can I encourage you to try something else, today?

Consider that for less than two Chipotle burritos (_with_ guac!) you can learn a new skill, study up on the fanciest new JS framework, try your hand at React Native or study some new domain you've always wondered about. 

With the AI coding assistants available today all of this is possible. But, better still: You can just build something.

What are you waiting for?

## An Idea

My son received a guitar for Christmas and it often goes out of tune. We have a analog chromatic tuner to keep it sounding nice, but since it is clunky and good for only one thing I usually have it tucked away somewhere. Digging for it is a pain because I don't remember where I put it and I don't want to lose it again. 

Yesterday, I wondered what it might take to make a Chromatic Tuner iOS app with Claude Code?

As it turns out, not very much.

## Multipurpose AI

Claude and other LLMs can build things, but they also can _teach_ you things. One thing it taught me was some of the science behind audio processing and chromatic analysis. An LLMs ability to teach concepts might be one of its most powerful features. 

## Get to the Point

My goal was to get to a working app without typing any code at all. First, I tried a one-shot prompt:

> Let's build an iOS app that listens to the user making a sound on a musical instrument, such as a guitar and determines what Chord (A, B, C, D, etc...) is being played. It shows the chord it found and all the relevant audio data on the screen for the moment the user is recording Audio. This app is for reference purposes and should be simple to use. It should use SwiftUI and good abstractions for Dependencies. Let's start with a plan to guide how this app will work. When you are done, we should be able to start the app, play a chord on a guitar and see the correct chord displayed on the app.

## One Shot Results

Without any adjustments to the UI this is what Claude created:

![Demo](/assets/demo.mov)

1. Clean UI with correct understanding of recording/not recording.
2. Histogram showing the audio buffer and each bin's frequency.
3. When a frequency roughly matches a known chord it is highlighted.
4. When an exact chord was detected it showed that chord.

## High-Level Summary

When the record button is tapped buffered audio data is sent through a [Fast Fourier Transform (FFT)](https://en.wikipedia.org/wiki/Fast_Fourier_transform) to convert it to a spectrum of frequencies. Frequencies are analyzed to determine "peaks". Peaks are estimated as muscal pitches. Frequencies within the musical range are kept and others are discarded. Peaks are then mapped to a [chromagram](https://en.wikipedia.org/wiki/Chroma_feature) which provides a rough mapping of which cord is represented by the frequency C, C#, D and so on.

## Where to go from here?

From this base we can easily tweak any of this code with targeted prompts. We can add features like Hz calibration, change the frequency visualization and add +/- guage to really see how much a tone is off. But that's for next time.