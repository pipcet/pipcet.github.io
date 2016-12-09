---
layout: post
title:  Dynamic Linking III
date:   2016-12-09 11:00:00 +0000
categories: wasm
---
I just compiled a dynamically-linked binary that looks like it should:
```
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <stdio.h>

int g(void)
{
  fprintf(stderr, "bra: %p\n", __builtin_return_address(0));

  return g();
}

int main(void)
{
  for (;;) {
    write(1, "hi\n", 3);
    int count = fprintf(stderr, "hey there: %p %p\n", stdout, stderr);
    g();
    break;
  }
}
```

It still isn't being run as it should be: we jump directly to `main` rather than initializing the C library and running constructors first ([musl][musl] appears to be very forgiving of that sort of thing, though).

The return address, while not as meaningful as it is on stored-program computers (i.e. you can't dereference it to look at the code), should be good enough for exception handling.

No strange compile flags, the library's loaded automatically, copy relocations work, fixup relocations distinguish code and data references.

Multiple libraries won't work yet.

The binary's 5111 bytes, less than the x86_64 ELF binary.

Unfortunately, `--no-threads` is still required until [bug 1322681][bug 1322681] is fixed.

[musl]: https://www.musl-libc.org/
[bug 1322681]: https://bugzilla.mozilla.org/show_bug.cgi?id=1322681
