---
layout: post
title:  Dynamic Linking VI
date:   2016-12-15 11:00:00 +0000
categories: wasm
---
So all the special cases we have for dynamic linking are getting a bit unwieldy. I think the *basic* pattern is this (FP means "function pointer"):

|reference| PIC code | non-PIC code |
|---|---|---|
|local call| PLT | direct |
|local FP | GOT | direct |
|extern call | PLT | PLT (→ direct)|
|extern FP | GOT | GOT |
|data | GOT | direct (→ copy reloc) |

There's one optimization opportunity that we're not taking: when linking non-PIC code, external function pointers that resolve to symbols in the same object file don't need to go through the GOT. However, that seems a relatively cheap thing, and fixing it would require a rather nasty relocation...

So we won't do that for now.

Still thinking about asm statements vs GCC attributes for synchronous (e.g. [Emscripten][emscripten]) functions.

Some more performance measurements on [bug 1253952][bug 1253952]: the performance gain is rather more significant than I thought, but offset by longer compile time, which is probably because my code isn't very well-written. I'll try simplifying my patch more and reducing it to the most significant performance-improvement bits.

Various minor bugs.

[emscripten]: https://github.com/kripken/emscripten
[bug 1253952]: https://bugzilla.mozilla.org/show_bug.cgi?id=1253952