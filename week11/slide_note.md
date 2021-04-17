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
# Of Linkers and Loaders
- 
