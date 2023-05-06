---
title: "AI-powered local interview transcription with Whisper"
date: 2023-05-06
slug: "whisper-ai-local-transcribe"
tags: ["research"]
author: "Philip Heltweg"
description: "Transcribing interviews is tedious, let AI do the work."
---

# The problem

Interviews are great. You can learn a lot about a topic by talking to experts. If you are working on a product, talking to users is common advice. If you are a researcher, qualitative surveys using interviews are great fun.

Transcribing interviews, however, sucks

Doing it yourself is tedious and is a painfully acquired skill. Outsourcing work to one of the various transcription APIs also is no perfect solution. Either you feed the beast that is large-scale AI models or pay even more money to trust Google, Amazon and co to not use your data.

# AI-powered local transcription

[Whisper](https://openai.com/research/whisper), recently released as open-source software by OpenAI, sounds like an alternative. It promises to be able to transcribe multiple languages, locally on your own workstation. Is the quality usable? It turns out, with a bit of setup, the answer is yes.

I conducted some interviews using the following process and transcribed them locally using Whisper. From start to finish, the transcription takes me roughly twice the interview time (so 120 mins for a 60 mins interview), half of which is my poor laptop struggling to auto-transcribe the interview.

As input, the setup uses separate audio files for the interviewer and participant. These can be created using Zoom and selecting "Record a separate audio file for each participant" in Settings, Recording.

To decrease random noise, microphones should be muted when not speaking. In video calls, nodding is best to encourage participants to continue to talk - you'll need to delete less "Yeah"s this way.

# The Code

All code for the setup is released in the GitHub repository [rhazn/whisper-local-transcribe](https://github.com/rhazn/whisper-local-transcribe). A description of the process and working notebook can be found there as [transcribe.ipynb](https://github.com/rhazn/whisper-local-transcribe/blob/main/transcribe.ipynb).

After automatically transcribing the interview, some manual cleanup is still needed. Helpful tools for this are [VLC](https://www.videolan.org/vlc/) to listen to the audio with [Global Hotkeys](https://wiki.videolan.org/VLC_HowTo/Global_hotkeys/) configured to pause/play and skip ahead/back 10s.

{{< aboutme >}}
