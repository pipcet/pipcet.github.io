---
layout: post
title:  Dynamic Linking
date:   2016-12-08 10:00:00 +0000
categories: wasm
---
I've been working on getting dynamic linking to work using my binutils/gcc asm.js/wasm backend at [https://github.com/pipcet/asmjs][asmjs]. My main objective is to make sure that [WebAssembly][wasm] has all the necessary bits to make it work, and that [SpiderMonkey][sm] has enabled them.

So far, things are looking good. I'm able to build a libc.so from the [musl][musl] sources (glibc isn't working yet), compile a "hello world" executable (with some annoying but currently-necessary flags), and run it, and it works.

### Main executable source code

```
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <stdio.h>

asm(".include \"wasm32-import-macros.s\"");

asm(".import3 thinthin,dlopen,__thinthin_dlopen\n"
    "\t.import3 thinthin,dlsym,__thinthin_dlsym\n");

extern int __thinthin_dlopen(const char *path) __attribute__((stackcall));
extern void *__thinthin_dlsym(const char *sym) __attribute__((stackcall));

asm(".import3 thinthin,syscall,__thinthin_syscall");

extern long __thinthin_syscall(long n, ...) __attribute__((stackcall));

long long __wasm_syscall(long long n, long long a1, long long a2, long long a3, long long a4, long long a5, long long a6) __attribute__((stackcall));


long long __wasm_syscall(long long n, long long a1, long long a2, long long a3, long long a4, long long a5, long long a6) {
  return __thinthin_syscall(n, a1, a2, a3, a4, a5, a6);
}
int main(void)
{
  __thinthin_syscall(1, 1, "hi0\n", 4);
  __thinthin_dlopen("libc.wasm");
  __thinthin_syscall(1, 1, "hi1\n", 4);
  for (;;) {
    int count = fprintf(stderr, "hey there: %p %p\n", stdout, stderr);
    break;
  }
}
```

`__thinthin_dlopen` and `__thinthin_dlsym` are syscall routines to access the (non-standard API) `ThinThin.dlopen` and `ThinThin.dlsym` JavaScript routines.

`__wasm_syscall` is defined in the executable, but used from `libc.so`/`libc.wasm`, just like `main` should be (but currently isn't). It simply performs a Linux-like syscall.

`main` calls `dlopen` explicitly (that's currently required; it would be nice to translate the `DT_NEEDED` entries in the `.dynamic` ELF section to automatically figure out which dynamic libraries to open, but that requires some tedious work to make sure we're not trying to open `libc.so`, which would be an ELF file); it then calls `fprintf`, which is defined in the dynamic library but referred to from the main executable, and ultimately ends up calling `__wasm_syscall` and printing the value of two data symbols defined in `libc.so`.

### Compiling the main executable

Currently, the invocation to build the test program is:

```
$ (cd tests; wasm32-virtual-wasm32-gcc -fPIC ./166-hello-world-dircall.c ../build/wasm32/musl/lib/libc.so -Wl,--warn-unresolved)
/home/pip/git/asmjs/wasm32-virtual-wasm32/lib/gcc/wasm32-virtual-wasm32/7.0.0/../../../../wasm32-virtual-wasm32/bin/ld: warning: type and size of dynamic symbol `stdout' are not defined
/home/pip/git/asmjs/wasm32-virtual-wasm32/lib/gcc/wasm32-virtual-wasm32/7.0.0/../../../../wasm32-virtual-wasm32/bin/ld: warning: type and size of dynamic symbol `stderr' are not defined
/home/pip/git/asmjs/wasm32-virtual-wasm32/lib/gcc/wasm32-virtual-wasm32/7.0.0/../../../../wasm32-virtual-wasm32/bin/ld: warning: cannot find entry symbol _start; not setting start address
/home/pip/git/asmjs/wasm32-virtual-wasm32/lib/gcc/wasm32-virtual-wasm32/7.0.0/../../../../wasm32-virtual-wasm32/bin/ld: /tmp/ccpTc2j6.o(.wasm.payload.code+0x132c): unresolvable R_ASMJS_LEB128_GOT relocation against symbol `stderr'
/home/pip/git/asmjs/wasm32-virtual-wasm32/lib/gcc/wasm32-virtual-wasm32/7.0.0/../../../../wasm32-virtual-wasm32/bin/ld: /tmp/ccpTc2j6.o(.wasm.payload.code+0x1360): unresolvable R_ASMJS_LEB128_GOT relocation against symbol `stdout'
/home/pip/git/asmjs/wasm32-virtual-wasm32/lib/gcc/wasm32-virtual-wasm32/7.0.0/../../../../wasm32-virtual-wasm32/bin/ld: /tmp/ccpTc2j6.o(.wasm.payload.code+0x1394): unresolvable R_ASMJS_LEB128_GOT relocation against symbol `stderr'
../build/wasm32/musl/lib/libc.so: warning: undefined reference to `longjmp'
../build/wasm32/musl/lib/libc.so: warning: undefined reference to `setjmp'
../build/wasm32/musl/lib/libc.so: warning: undefined reference to `__wasm_blocks___one_cmplsi2'
../build/wasm32/musl/lib/libc.so: warning: undefined reference to `__wasm_ast___one_cmplsi2'
```

Some of those warnings look fatal, but they don't actually appear to be. Currently both `-fPIC` (to create GOT relocs for data accesses) and `-Wl,--warn-unresolved` (to avoid fatal errors for `longjmp` and `setjmp`, which genuinely aren't defined yet; and to avoid the other fatal errors, which should be fixable another way) are required.

### Compiling the shared library

```
$ make build/wasm32/musl.make
$ bash -x ./bin/wasmify-wasm32 build/wasm32/musl/lib/libc.so > libc.wasm
```

### Running

```
$ tests/a.out
wasm32-virtual-wasm32-ld: warning: cannot find entry symbol _start; defaulting to 0000000000000000
calling pc0 9 dpc undefined rpc 0 sp 1072691792
hi0
dlopen
dlopen: 16553 libc.wasm
dlopen: [object Object]
called: 1072691649
calling pc0 5 dpc 0 rpc 0 sp 1072691648
called: 1072691649
calling pc0 5 dpc 0 rpc 0 sp 1072691648
called: 1072691649
calling pc0 5 dpc 0 rpc 0 sp 1072691648
called: 1072691649
calling pc0 5 dpc 0 rpc 0 sp 1072691648
called: 1072691649
calling pc0 5 dpc 0 rpc 0 sp 1072691648
called: 1072691649
success
calling pc0 5 dpc 0 rpc 0 sp 1072691648
called: 1072691649
calling pc0 5 dpc 0 rpc 0 sp 1072691648
called: 1072691649
calling pc0 5 dpc 0 rpc 0 sp 1072691648
called: 1072691704
calling pc0 9 dpc 0 rpc 0 sp 1072691744
hi1
hey there: 0x20001d10 0x20001be0
called: 1072691792
calling pc0 0 dpc 0 rpc 0 sp 0
another exception
TypeError: this.vm.table.get(...) is not a function
Wasm32Thread.prototype.indcall@/tmp/tmp.KCsO4oBKVZ:552:12
Wasm32Thread.prototype.step@/tmp/tmp.KCsO4oBKVZ:508:19
f@/tmp/tmp.KCsO4oBKVZ:2923:17
```

Lots of debug output there, and some fatal errors at the end (this is because we're currently using `main` as entry point rather than `_start`, which never returns. When `main` does, we run into trouble). But it works! `dlopen` resolves symbols (both those defined in the executable and those defined in the shared library), and GCC emits the right code as long as `-fPIC` is specified.

## Why `-fPIC`? Lack of copy relocations

The most vexing problem right now is the requirement for the executable to be compiled with `-fPIC`, even though it's ordinarily only the shared library that requires this flag. The problem is the lack of [copy relocations][copy-relocations], which I hadn't known about.

## The `dyninfo` custom section

We're not trying to parse the ELF data at runtime, since the intended usage is to distribute only a wasm file, but we need access to dynamic relocations, dynamic symbols, and some additional data for the JavaScript dynamic loader to access at runtime. So what we're doing right now is to read that information from the ELF file, run it through a Perl script, and produce a wasm custom section containing that information in JavaScript notation (right now it's run through `eval`, but should switch to using `JSON` soon).

That section is called `dyninfo`, and constructs a single JavaScript object with the following members:

- `ref`: dynamic references to data/data relocations
- `refun`: dynamic references to code/code relocations
- `def`: dynamic symbols defining data
- `defun`: dynamic symbols defining code
- `fixup`: references to local data
- `fixupfun`: references to local code
- `plt_bias`, `plt_end`: limits of the PLT
- `data`, `data_end`: size and offset of the data section

## Function pointers in data sections

Another problem we're currently running into is that function pointers can be put into data sections; but function indices and data pointers live in separate spaces in wasm, with different relocation requirements. I'm currently considering adding an extra `R_ASMJS_ABS32_CODE` relocation type to catch this. That would rely on gcc to detect this situation and generate a special reloc, probably by explicitly using the `.reloc` assembler directive.

[asmjs]: https://github.com/pipcet/asmjs
[wasm]: https://WebAssembly.github.io
[sm]: https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey
[copy-relocations]: http://docs.oracle.com/cd/E19683-01/817-3677/6mj8mbtbs/index.html#chapter4-84604
[musl]: https://www.musl-libc.org/
