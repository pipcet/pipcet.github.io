---
layout: post
title:  Dynamic Linking
date:   2016-12-08 05:55:00 +0000
categories: wasm
---
I've been working on getting dynamic linking to work using my binutils/gcc/glibc asm.js/wasm backend at [https://github.com/pipcet/asmjs][asmjs]. My main objective is to make sure that [WebAssembly][wasm] has all the necessary bits to make it work, and that [SpiderMonkey][sm] has enabled them.

[asmjs]: https://github.com/pipcet/asmjs
[wasm]: https://github.com/WebAssembly/design
