---
layout: post
title:  Dynamic Linking IV
date:   2016-12-11 06:00:00 +0000
categories: wasm
---
So far my dynamic linking experiments have been based on [musl]. Apart from being significantly smaller than [glibc], it was quite easy to get musl to build on the wasm32 architecture: copy the x86_64 customization, edit a few files, done.

That's not true of glibc. It was hard enough to get glibc to build statically, but creating a dynamic library is much harder, partly because glibc appears to assume I want to use an ELF-based dynamic loader; we can't do that, because then we'd have to send ELF files rather than wasm objects, put them into memory, relocate them, then convert them to wasm format anyway for evaluation.

So I don't want `ld.so`, just `libc.so`. That appears to be very hard. Fixing things so `ld.so` builds also isn't precisely easy. I'm not quite at the point of giving up and using the static library's build process with `-fPIC`, but it's beginning to seem like the most attractive option.

[musl]: https://www.musl-libc.org/
[glibc]: https://www.gnu.org/software/libc/
