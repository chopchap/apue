# slide notes
---
## an illustration of a process in memory
![an illustration of a process in memory](https://github.com/chopchap/apue/blob/main/images/an%20illustration%20of%20process%20in%20memory.png?raw=true)
- `pmap(1)/pmap(9)` The pmap utility lists the virtual memory mappings underlying the given process.
## program start
- The name 'main' and whatever prototype it may have, after all, is a programming language specific thing.
- When one of the 'exec' functions is called, the kernel calls a special startup routine, which sets up the process space, puts the command-line arguments and the environment into the right place etc.
- compiling it with '-g' to enable debugging symbols
- inside gdb debugging, change layout by typing `layout reg`, to show us both the common registers as well as the assembly of the executable.
- `s` in gdb means to single-step.
- `p val` in gdb means to print the value of val
- `x/s $rdi` in gdb means to print hex contents of register `rdi` (i.e. the argument register)
- `p/s $rsi` in gdb means to print the second argument's value as hex (`rsi` is the second argument register by convention)
- by using gdb, we found that our program didn't really start at 'main'.  It started in something called '_start'. By looking at the source tree, the file /usr/src/lib/csu/arch/x86_64/crt0.S (csu stands for _C Start Up_; crt stands for _C Runtime_) contains the initial entry point of the program, the '_start' assembly routine, which calls the '___start' function. That '___start' function is a regular C function found in the crt0-common.c file. It takes two arguments, a function pointer, and a struct ps_strings. This struct ps_strings is found at the top of the stack and is used to locate the arguments and environment strings. So the '___start' function uses these strings to set up the 'extern char **environ' as well as the __progname string, which is used by the `getprogname(3)` library function. Further down, we see the calls to 'atexit' and the _libc_init function. Finally, it calls 'exit', passing it the return value of 'main', to which it passes the argc, argv, and environment pointers.
- `i frame` in gdb to inspect the frame
- `break *addr` in gdb to set a breakpoint at the given address
- `rip` register saves the returning addr in the stack.
- `cc -e funcname -g file.c` can set the entry function of a compiled program
- takeaways:
  - the entry point is defined by the compiler / linker. Meaning we can change it.
  - there is a C Startup routine that sets up the environment and moves the command-line arguments into place etc.
  - we saw why 'main' always returns an int: it passes this value on to 'exit'.
  - when a program is started, it follows the flow: `_start() -> ___start() -> exit(main(...))`
- all in all, our program doesn't simply start with 'main', but instead we notice the default entry point '_start', which calls '___start', which calls 'exit' with its argument being the return value of 'main'.
## 
