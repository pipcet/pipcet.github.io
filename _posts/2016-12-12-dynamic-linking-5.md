---
layout: post
title:  Dynamic Linking V
date:   2016-12-14 11:00:00 +0000
categories: wasm
---
Okay, I've decided to go with the approach of building `libc.a` with `-fPIC`, then relinking that into `libc.so`. That seems to work!

The precise invocation is

```
$ wasm32-virtual-wasm32-ld -shared -o libc.so --whole-archive -'(' build/wasm32/glibc/libc.a   -')' --no-whole-archive ./wasm32-virtual-wasm32/lib/gcc/wasm32-virtual-wasm32/7.0.0/libgcc.a
$ bash -x ./bin/wasmify-wasm32 ./libc.so > libc.wasm
```

I've experimented a little, and compile time for the wasm binaries is a lot shorter than for their asm.js equivalents. They're obviously a lot smaller, but, surprisingly to me, they're also significantly smaller once compressed with 7zip.

A lot of performance measurements, including with the patch from [bug 1253952][bug 1253952]. That patch appears to have much less of an effect on wasm than it did on asm.js, and I'm not sure why—the generated code looks right in both cases.

Tried building the build/wasm32/perl.make target from scratch on a VM. It works, if you run `make` targets in the right order.

I think wasm custom sections names should be URIs, except when a short name has been made official. [Tag URIs][tag uri scheme] allow anyone with an email address to mint unique URIs that are explicitly not resolvable, so it's not a restriction.

Thinking about how to interface with synchronous wasm functions. What should Just Work is asm statements, but it would be nice to write something like

```
extern double emscripten_sqrt(double) __attribute__((synccall));
```

and be able to use `emscripten_sqrt` as an ordinary function. Maybe even a function pointer. Hmm.

Some misguided (and since-reverted) changes to the GOT stuff.

`-Os` is broken. [Glibc][glibc] uses magic to force that function to be statically linked in dynamic libraries, and that appears not to work with our semi-shared approach.

`-O2` is significantly faster than `-O3`. Maybe we're running out of icache? Weird, `-O1` is even faster.

[musl]: https://www.musl-libc.org/
[glibc]: https://www.gnu.org/software/libc/
[tag uri scheme]: https://tools.ietf.org/html/rfc4151
[bug 1253952]: https://bugzilla.mozilla.org/show_bug.cgi?id=1253952