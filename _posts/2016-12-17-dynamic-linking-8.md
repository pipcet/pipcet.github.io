---
layout: post
title:  Dynamic Linking VIII
date:   2016-12-16 11:00:00 +0000
categories: wasm
---
Things are progressing well: dynamically-loaded Perl modules appear to be working, and there's a `libdl.wasm` now to handle `dlopen` and the like.

@rossberg-chromium gave a [good summary](https://github.com/WebAssembly/design/issues/910#issuecomment-267297801) of the ABI used for dynamic linking. Slightly modified, it reads:

 - There is a central `malloc` implementation that manages the linear memory, and the loader communicates with it
 - All other modules only acquire memory dynamically by going through `malloc`
 - All data segments except for the main program's use an imported global for base address
 - No memory address is hard-coded in any module except for the main program's
 - No module defines its own linear memory.

All those seemed kind of obvious to me, but spelling them out explicitly is important. In more detail:

 - every dynamically-loadable module imports three globals: `sys.got`, `sys.plt`, and `sys.gpo`.
 - `sys.got` points at sufficient memory for the module's data segment in the imported linear memory
 - `sys.plt` is the start of a range of table indices for the modules' use, in the imported function table
 - `sys.gpo` is the start of a range of "PC values" for the module to use. This is totally independent of wasm.
 - every module has a `dyninfo` section containing:
 -- a list of defined symbols and their positions
 -- a list of required symbols and their PLT offsets or GOT positions
 -- a list of fixup relocations, memory addresses that contain pointers to the data segment that need to be fixed
 -- a list of library dependencies
 -- the size of the data segment
 -- the number of function table entries
 -- the size of the PC range
