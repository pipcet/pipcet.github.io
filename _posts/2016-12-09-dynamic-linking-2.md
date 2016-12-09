---
layout: post
title:  Dynamic Linking II
date:   2016-12-09 11:00:00 +0000
categories: wasm
---
Some progress: I've decided to change the calling convention of wasm functions slightly, in that the last two arguments ("pc0" and "rpc") are now ignored, and calculated when needed by the callee.  I haven't actually dropped them yet.

I just noticed that dynamic linking with SpiderMonkey is broken right
now unless you specify `JFLAGS=--no-threads`. The problem is that we never hand control back to the VM in a way sufficient for it to finish instantiating our wasm module.

There is now a program counter space to go with the linear memory space and the function index space; this should allow C++ exceptions to work again at some point.

Copy relocations now work.

The strange compile flags I mentioned yesterday, `-Wl,--warn-unresolved` and `-fPIC` (for the main executable) are no longer required.

The explicit call to `dlopen` is still required.

I've added the `R_ASMJS_ABS32_CODE` relocation to handle function pointers in data sections, but haven't implemented that yet.
