---
layout: post
title:  Dynamic Linking VII
date:   2016-12-16 11:00:00 +0000
categories: wasm
---
Here's a slightly tricky bit about dynamic linking relocations that I keep forgetting: PLT entries jump to their own function space index in the function table, which is set up to contain the function table entry corresponding to the remote function.

I've been working on reducing the number of `R_ASMJS_NONE` relocations we were leaving behind because we sized the relocation sections too generously.
