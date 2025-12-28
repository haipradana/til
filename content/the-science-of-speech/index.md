---
title: "The Science of Speech"
date: 2025-12-28T22:10:12+07:00
draft: false
tags:
    - Neuroscience

# Author
language: en
---

Conversation is feedback loop. We monitor our own speech as we generate it. 

It's like how context works in LLM. We initially process based on the input, then as we generate the output, we add the output token to the context window.

The process starts in **Heschl's gyrus** (HG) in the auditory cortex, located in temporal lobe. The sound waves are converted to electrical signals by the cochlea in the ear (encoder) and transmitted to HG. 

Then the auditory cortex analyzes the spectrotemporal features (let's say it sound metadata)---the pitch, the loudness, and the duration.

Next to this area is **Wernicke's Area**, whose role as ***phonological lexicon***, mental storage of words, consisting of sound structure (pitch, stress, rhytm) and how they interact with grammar for building new words. We can distinguish between "bat" and "cat" by sound with this. It's like a **tokenizer** in LLM included with lexer and parser.

Then, in the semantic network, single concept "apple" is not stored in one neuron, but it's connection between multiple abstract associations ans sensory. There is **The Angular Gyrus** in the Parietal Lobe, it binds the word "apple" to the visual (the shape of fruit), the sensation, into a package.