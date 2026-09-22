<style>
.reveal .slides section .fragment.highlight-code-red {
    opacity: 1;
    visibility: inherit;
}
.reveal .slides section .fragment.highlight-code-red.current-fragment {
    background-color: orangered;
}
.reveal .slides section .fragment.highlight-code-bold {
    opacity: 1;
    visibility: inherit;
}
.reveal .slides section .fragment.highlight-code-bold.current-fragment {
    font-weight: bold;
    background-color: #17ff2e;
}
.hljs-variable {
    color: darkgoldenrod;
}
.hljs-literal {
  color: purple;
}
</style>
<div id="markdown-custom-center">Linux Shared Libraries</div>
<!-- .slide: class="center" style="font-size:80%; padding-left:20%; margin-top: -1%; width:80%;" data-background-transition="slide" data-background-image="images/ppt-arno-1.png" data-state="hide-footer" -->

# Linux shared libraries
## Arno Lepisk
### <code style="background: inherit;">arno@lepisk.se</code> 
### 2026-09-24

---
# Agenda
* What is a shared library
* How does one use shared libraries
  * Which tools exist
* How does one create shared libraries
* What is happening behind the scenes

???

We'll jump back and forth between these

Ask questions - otherwise this might be short

---

# What is a shared library?

Reusable code - your application share libraries 
with other applications

In Linux they're commonly called libXXX.so.N,
but can be called anything

so - shared object

(And everything called .so isn't necessarily a library)

???

I use shared and dynamic library interchangeably.

There is technically a difference but not important in this context 

--
# 🛠️ `ldd`
A useful tool which shows which dynamic libraries are loaded by an
application

```text
$ ldd main
    linux-vdso.so.1 (0x00007ffdf7f2d000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f8f518ef000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f8f51ad8000)
```

???

might "lie"

---
# How one uses a shared library

```text
$ gcc -o outfile object.o -l library-without-lib
```

```text
$ gcc -o mainz mainz.o -lz
```
alt
```text
$ gcc -o mainz mainz.o /lib/x86_64-linux-gnu/libz.so
```

--
# Check with `ldd`

<pre class="text"><code data-trim data-noescape>
$ ldd ./mainz
    linux-vdso.so.1 (0x00007ffdd93f0000)
    <b>libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007fce656a9000)</b>
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fce654d5000)
    /lib64/ld-linux-x86-64.so.2 (0x00007fce656db000)
</code></pre>

???

But where does .1 come from?

--
# What's in the filesystem?

```text
$ ls -l /lib/x86_64-linux-gnu/libz.*
/lib/x86_64-linux-gnu/libz.a
/lib/x86_64-linux-gnu/libz.so -> /lib/x86_64-linux-gnu/libz.so.1.2.11
/lib/x86_64-linux-gnu/libz.so.1 -> libz.so.1.2.11
/lib/x86_64-linux-gnu/libz.so.1.2.11
```

???

how does the system know what to use?

--
# 🛠️ `readelf -d`
a tool that reads information from ELF-files

<pre class="text"><code data-trim data-noescape>
$ LANG=C readelf -d /lib/x86_64-linux-gnu/libz.so
...
 0x0000000000000001 (NEEDED)      Shared library: [libc.so.6]
 0x000000000000000e (SONAME)      <b>Library soname: [libz.so.1]</b>
...
</code></pre>


???

explain ELF

--
# Common distribution

Versioned libraries are distributed, where SONAME is set to the compatible version
(ie major)

In the development package a unversioned file is present, pointing to the latest version.

--
# Symbol versioning

For backward compatibility purposes symbols within the library can be versioned as well,
commonly encountered by errors like
```text
$ ./main 
./main: /lib/x86_64-linux-gnu/libstdc++.so.6: 
      version `GLIBCXX_3.4.32' not found (required by ./main)
./main: /lib/x86_64-linux-gnu/libc.so.6: 
      version `GLIBC_2.34' not found (required by ./main)
```

(won't be covered further in this talk)

--
# A .so-file doesn't need to be a binary!
```text
$ cat /usr/lib/x86_64-linux-gnu/libm.so
/* GNU ld script
*/
OUTPUT_FORMAT(elf64-x86-64)
GROUP ( /lib/x86_64-linux-gnu/libm.so.6  AS_NEEDED 
        ( /lib/x86_64-linux-gnu/libmvec.so.1 ) )
```

--- 
# How to create your own?
Back to basics
```text
$ g++ -o file1.o -c file1.cpp
$ g++ -o file2.o -c file2.cpp
...
$ g++ -o program file1.o file2.o ...
```
--
# A somewhat larger program
```text
$ g++ -o file1.o -c file1.cpp
...
$ ar r mylib.a file1.o ...
...
$ g++ -o program filex.o mylib.a
```

???

static lib

--
# Ok? What about a shared library?
```text
$ g++ -o file.o -c file.cpp
$ g++ -o mylib.so -shared fil.o
/usr/bin/ld: fil.o: relocation R_X86_64_PC32 against symbol 
     `_ZSt4cout@@GLIBCXX_3.4' can not be used when making a 
     shared object; recompile with -fPIC
```

Hepp.

Luckily the error message tells us what to do

--
# `-fPIC`
Position Independent Code

The binary can execute wherever it is placed in memory

Uses relative addressing

??? 

Closely related: PIE - Position independent executable

--
# What about now?
```text
$ g++ -o file.o -c file.cpp -fPIC
$ g++ -o mylib.so -shared file.o
```
# Use it
```text
$ g++ -o main.o -c main.cpp
$ g++ -o main main.o mylib.so
```
--
# And run!
```text
$ ./main
./main: error while loading shared libraries: mylib.so: 
        cannot open shared object file: No such file or directory
```
<p class="fragment">😢<br/>meh</p>

--
# `LD_LIBRARY_PATH`

One way of telling the dynamic linker of where to find libraries
```text
$ LD_LIBRARY_PATH=. ./main
Hello world!
```

-- 
# `-rpath`
```text
$ g++ -o main main.o mylib.so -Wl,-rpath=\$ORIGIN
$ ./main
Hello world!
```
Another way of encoding into the binary where to search for libraries.

`$ORIGIN` is a special "placeholder" to search relative to the binary itself. 

When crosscompiling `-rpath-link` can come in useful.

???

Commandline says rpath, actually sets runpath.

rpath is legacy and differs in search order priority in regard to LD_LIBRARY_PATH

---
# ⚠️ It's just code!

The code itself doesn't care.

In essence, it's about how one distributes ones binaries.

---
# Another way of using libraries - `libdl`

`libdl` is used to programmatically load libraries at runtime.
`#include "dlfcn.h"`
* `dlopen` - open the library
* `dlsym` - fetch a symbol
* `dlclose` - close the library

Could be used to implement a plugin system.

--
# example - runner
```c++
int main(int argc, char **argv) {
  for(int i = 1; i < argc; i++) {
    if(auto hndl = dlopen(argv[i], RTLD_NOW|RTLD_LOCAL); hndl != nullptr) {
      if(auto fh = dlsym(hndl, "plugin_entry"); fh != nullptr) {
        auto func = reinterpret_cast<void(*)()>(fh);
        func();
      } else {
        std::cout << "failed to find plugin_entry in " << argv[i] << '\n';
      }
      dlclose(hndl);
    } else std::cout << "failed to load " << argv[i] << '\n';
  }
}
```
--
# example - plugin 1
```c++
#include <iostream>

void plugin_entry() {
  std::cout << "hello plugin 1" << '\n';
}
```
```text
$ g++ -o plugin1.so -shared -fPIC plugin1.cpp
$ ./runner ./plugin1.so 
failed to find plugin_entry in ./plugin1.so
```
<!-- .element class="fragment" -->
Wut? Did we spell something wrong? <!-- .element class="fragment" -->

--
# 🛠️ `nm` - lists symbols
```text
$ nm plugin1.so
...
0000000000001135 T _Z12plugin_entryv
...
```
```text
$ c++filt <<< _Z12plugin_entryv
plugin_entry()
```
<!-- .element class="fragment" -->

???

C++ name mangling

--
# attempt 2
```c++
#include <iostream>

extern "C" void plugin_entry() {
  std::cout << "hello plugin 1" << '\n';
}
```
```text
$ ./runner ./plugin1.so 
hello plugin 1
```

???

It's important to think about what ends up in the binary if one wants to read it by name.

---
# What happens behind the curtains?
## ELF - Executable and Linkable Format

As the program is about to start the kernel reads the file.
If it is a dynamically linked ELF-executable the kernel reads
info from it, maps memory and hands over to the dynamic linker.

How is the dynamic linker found?

--
# 🛠️ `readelf -l`
```text
$ LANG=C readelf -l main
Elf file type is DYN (Shared object file)
...
Program Headers:
...
  INTERP         0x00000000000002a8 0x00000000000002a8 0x00000000000002a8
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```
RPI:
```text
      [Requesting program interpreter: /lib/ld-linux-aarch64.so.1]
```
Old RPI:
```text
      [Requesting program interpreter: /lib/ld-linux-armhf.so.3]
```
--
# What does the linker do?
<pre class="text"><code data-trim data-noescape>
$ LANG=C readelf -d /lib/x86_64-linux-gnu/libz.so
...
 0x0000000000000001 (<b>NEEDED</b>)      Shared library: [libc.so.6]
 0x000000000000000e (SONAME)      Library soname: [libz.so.1]
...
</code></pre>

Reads which libraries are needed (recursively),
sets up symbol tables etc.

--
# Symbol table?
```text
$ nm -C main
...
0000000000001175 T main
...
                 U func()
$ nm -C mylib.so
...
0000000000001135 T func()
```
```text
T - text (code)
U - undefined
B - BSS data
D - data
R - Read only data
X/x - global/lokal
```
---
# LD_DEBUG, LD_TRACE_LOADED_OBJECTS
The dynamic linker has some built-in debug tools.
```text
$ LD_DEBUG=help ./main
Valid options for the LD_DEBUG environment variable are:

  libs        display library search paths
  reloc       display relocation processing
...
```
```text
$ LD_TRACE_LOADED_OBJECTS=1 ./main
        linux-vdso.so.1 (0x00007ffdb05a3000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fa97151c000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fa971700000)
```
same output as ldd
---
# stub-libs

A somewhat underused technique is stub-libs. 

Libraries with stub (empty) functions for all exposed functionality, but points out a 
different library with the actual functionality used at execution. 

As the stub library doesn't contain any actual functionality it becomes much smaller, which greatly 
can reduce times for eg CI

---
<!-- .slide: class="center" data-state="hide-footer" -->
# Thanks
## Arno Lepisk
### <code style="background: inherit;">arno@lepisk.se</code>

