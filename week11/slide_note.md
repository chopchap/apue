# slide notes
---
## The Executable & Linkable Format
- Compilers produce, and linkers and loaders opearte on _object files_. Just like other file, they have specific formats such as _e.g._, assembler output (a.out), Common Object File Format (COFF), Mach-O, or ELF.
  - executable - just what it sounds like (e.g. a.out)
  - core - virtual address space and register state of a process; debugging information (a.out.core)
  - relocatable file - can be linked together with others to produce a shared libaray or an executable (_e.g._, foo.o)
  - shared object file - position independent code; used by the dynamic linker to create a process image (_e.g._, libfoo.so)
- How an ELF file is structured?
  - ![how ELF is structured](https://github.com/chopchap/apue/blob/main/images/how%20ELF%20structured.png?raw=true)
    - An ELF file is made up of an ELF header followed by some data.  The data includes:
      - the program header table, containing entries that point at different memory segments, such as the data or text segments.
      - as well as the section header table, containing entries that point to selected _sections_
      - The segments pointed to from the program header table are used to provide information required at execution time, while the segments pointed to from the section header table are used at link time.
  - `$ hexdump -C main.o | head 2`
  - `$ readelf -h main.o`
## Of Linkers and Loaders
- The compiler chain:
  1. `$ cpp crypt.c crypt.i`
  2. `$ cc -S crypt.i`
  3. `$ as -o crypt.o crypt.s`
  4. `$ ld -dynamic-linker /usr/libexec/ld.elf_so /usr/lib/crt0.o /usr/lib/crti.o /usr/lib/crtbegin.o crypt.o -lcrypto -lc /usr/lib/crtend.o /usr/lib/crtn.o`
- In summary, the linker inserts the program interpreter, /usr/libexec/ld.elf_so, and then combines the object files with the shared libraries that we told it to link with to create the a.out ELF binary.
- `ktrace(1)`: ktrace enables kernel trace logging for the specified processes
- `kdump(1)`: display kernel trace data
- runtime linker:
  ![runtime linker](https://github.com/chopchap/apue/blob/main/images/runtime%20linker.png?raw=true)
## Shared Libraries
- How do shared libraries work?
  - at _link time_, the linker resolves undefined symbols
  - contents of object files and _static library_ are pulled into the executable at link time
  - contents of _dynamic libraries_ are used to resolve symbols at **link time**, but loaded at **execution time** by the _dynamic linker_
  - contents of _dynamic libraries_ may be loaded **at any** time via explicit calls to the dynamic linking loader interface functions
- `$ nm a.out`: list symbols from object files
- `$ ar -vq libldtest.a ldtest[12].o`: create, modify, and extract from archives (static library)
- `dlopen(3)`: dynamic link interface
