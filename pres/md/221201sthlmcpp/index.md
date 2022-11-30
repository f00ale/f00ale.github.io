<div id="markdown-custom-center">Sharing is Caring</div>
<!-- .slide: class="center" style="font-size:80%; padding-left:20%; margin-top: -1%; width:80%;" data-background-transition="slide" data-background-image="images/ppt-arno-1.png" data-state="hide-footer" -->
# Sharing is Caring
## Arno Lepisk
### <code style="background: inherit;">arno@lepisk.se</code> 
### <span class="twitter"><code style="background: inherit;">@arno_l</code></span> &nbsp; <span class="github"><code style="background: inherit;">f00ale</code></span>
### StockholmCpp 2022-12-01

---
# Shared libraries
* not exhaustive
* Linux-centric (should be roughly the same on ELF-based systems)

---
# Recap - how do we build binaries?
1. Compile and link in one step 
   - `g++ -o main main.cpp lib.cpp`
2. Compile and link in separate steps
   - `g++ -c -o main.o main.cpp`
   - `g++ -c -o lib.o lib.cpp`
   - `g++ -o main main.o lib.o`

---
# Larger projects - static libraries
- Put a bunch of object files together into an archive
  - `ar rcs libmylib.a lib.o ...`
- Use the library
  - `g++ -o main main.o libmylib.a`
  - `g++ -o main main.o -L. -lmylib`
- Leads to duplication if you have several libraries <!-- .element class="fragment" -->
- ... and large binaries <!-- .element class="fragment" -->
---
# Shared libraries
- Objects must be compiled with `-fPIC`:
  - `g++ -c -o obj.o -fPIC obj.cpp`
- Now you can build the library
  - `g++ -shared -o libmylib.so obj.o`
- And build your program
  - `g++ -o main main.o libmylib.so`
  - `g++ -o main main.o -L. -lmylib`
---
# Run it
```text
$ ./main
./main: error while loading shared libraries: libmylib.so: 
    cannot open shared object file: No such file or directory
```
- Dynamic linker cannot find library! <!-- .element class="fragment" -->

<pre class="fragment text"><code data-trim>
$ LD_LIBRARY_PATH=. ./main
...
</code></pre>
---
# `RPATH`/`RUNPATH`
* One can add path to look in via the `-rpath` argument to the linker
  * `g++ -Wl,-rpath=<path> -o main main.o -L. -lmylib`
* Special value: `$ORIGIN` - looks in the directory of the binary
  * `g++ -Wl,-rpath='$ORIGIN' -o main main.o -L. -lmylib`
* Linker decides if to use `RPATH` or `RUNPATH` - can be forced
  via `--enable-new-dtags`/`--disable-new-dtags`

---
# Library versions
One often sees libraries like this:
```text
libmylib.so      -> libmylib.so.1.23
libmylib.so.1    -> libmylib.so.1.23
libmylib.so.1.23
```
Linking against `libmylib.so` actually links against `libmylib.so.1` (checked via `ldd`) - how does that work?

---
# `SONAME`
SONAME is a mechanism to tell a library it's name. 
- `g++ -shared -Wl,-soname=libmylib.so.1 -o libmylib.so.1.23 obj.o ...`
- i.e create a library libmylib.so.1.23, but call it libmylib.so.1 internally.

---
# Stub libraries
This can be used for "stub"-libraries
- `g++ -shared -Wl,-soname=libreal.so.1 -o libstub.so ...`

Can have stub implementation of public interface, but much smaller
  (to minimize size of CI builders etc)

---
<!-- .slide: class="center" data-state="hide-footer" -->
## Arno Lepisk
### <code style="background: inherit;">arno@lepisk.se</code>
### <span class="twitter"><code style="background: inherit;">@arno_l</code></span> &nbsp; <span class="github"><code style="background: inherit;">f00ale</code></span>
