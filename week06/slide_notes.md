# slide notes
---
## an illustration of a process in memory
![an illustration of a process in memory](https://github.com/chopchap/apue/blob/main/images/an%20illustration%20of%20process%20in%20memory.png?raw=true)
- `pmap(1)/pmap(9)` The pmap utility lists the virtual memory mappings underlying the given process.
## program startup
- The name 'main' and whatever prototype it may have, after all, is a programming language specific thing.
- When one of the 'exec' functions is called, the kernel calls a special startup routine, which sets up the process space, puts the command-line arguments and the environment into the right place etc.
- compiling it with '-g' to enable debugging symbols
- inside gdb debugging, change layout by typing `layout reg`, to show us both the common registers as well as the assembly of the executable.
- `s` in gdb means to single-step.
- `p val` in gdb means to print the value of val
- `x/s $rdi` in gdb means to print hex contents of register `$rdi` (i.e. the argument register)
- `p/s $rsi` in gdb means to print the second argument's value as hex (`#rsi` is the second argument register by convention)
- by using gdb, we found that our program didn't really start at 'main'.  It started in something called '_start'. By looking at the source tree, the file /usr/src/lib/csu/arch/x86_64/crt0.S (csu stands for _C Start Up_; crt stands for _C Runtime_) contains the initial entry point of the program, the '_start' assembly routine, which calls the '___start' function. That '___start' function is a regular C function found in the crt0-common.c file. It takes two arguments, a function pointer, and a struct ps_strings. This struct ps_strings is found at the top of the stack and is used to locate the arguments and environment strings. So the '___start' function uses these strings to set up the 'extern char **environ' as well as the __progname string, which is used by the `getprogname(3)` library function. Further down, we see the calls to 'atexit' and the _libc_init function. Finally, it calls 'exit', passing it the return value of 'main', to which it passes the argc, argv, and environment pointers.
- `i frame` in gdb to inspect the frame
- `break *addr` in gdb to set a breakpoint at the given address
- `$rip` register saves the returning addr in the stack.
- `cc -e funcname -g file.c` can set the entry function of a compiled program
- takeaways:
  - the entry point is defined by the compiler / linker. Meaning we can change it.
  - there is a C Startup routine that sets up the environment and moves the command-line arguments into place etc.
  - we saw why 'main' always returns an int: it passes this value on to 'exit'.
  - when a program is started, it follows the flow: `_start() -> ___start() -> exit(main(...))`
- all in all, our program doesn't simply start with 'main', but instead we notice the default entry point '_start', which calls '___start', which calls 'exit' with its argument being the return value of 'main'.
## program termination
- `objdump -d a.out`: Display the assembler mnemonics for the machine instructions from objfile
- there're multiple ways to terminate a process (In most cases you want to simply call exit(3) to ensure proper cleanup takes place.):
![multiple ways to terminate a process](https://github.com/chopchap/apue/blob/main/images/multiple%20ways%20to%20terminate%20a%20process.png?raw=true)
- exit(3) and _exit(2)
  - exit(3) terminates a process. Before termination it performs the following functions in order listed:
    - call the functions registered with the `atexit(3)` function, in the reverse order of the their registration.
    - flush all open output streams
    - close all open streams
    - unlink all files created with the tmpfile(3) function.
    - call _exit(2)
  - _exit(2) terminates the process immediately
- lifetime of a Unix process
![lifetime of a Unix process](https://github.com/chopchap/apue/blob/main/images/lifetime%20of%20a%20Unix%20process.png?raw=true)
## the environment
- `$env(1)` utility can print the current environment
- setting variables in one process does not have any effect on another process, so the environment is process specific.
- getenv(3) & putenv(3) & setenv(3) & unsetenv(3)
- extern char **environ (global) & char **envp (local to 'main')
  - `envp`, as an argument to 'main' must be on the stack.
  - setting variables in the environment via setenv(3) / putenv(3) may lead to shuffling some or all elements of the environ to dynaically allocated memory, but `envp`  doesn't get updated.
## process limits and identifiers
- `ulimit -a` to list resources that we can inspect
- a `shell builtin` is a command or a function, called from a shell, that is executed directly in the shell itself, instead of an external executable program which the shell would load and execute. (e.g. ulimit, cd etc.)
- ulimit is obselete, instead, it calls getrlimit(3) and setrlimit(3)
- A soft limit may be _lowered_ by a process or _raised_ to a value lower than or equal to the _hard_ limit. and changing the limit also set the new hard limit.
- `echo $$` to print the pid of shell
- `ps -waxo pid,ppid,command` or `proctree` to print snapshot of processes alive
## process control
- fork(2): 
  - the child process has its own copy of the parent's descriptors.
  - no order of execution child and parent is not guaranteed.
  - sharing of open files between child and parent after fork
  ![sharing of open files between child and parent after fork](https://github.com/chopchap/apue/blob/main/images/sharing%20of%20open%20files%20between%20parent%20and%20child%20after%20fork.png?raw=true)
  - that means that an operation on either of these file descriptors in one process affects the other.
  - when stdout is connected to a pipe, it no longer is line buffered. That is, we keep the data in the buffer without flushing the buffer. Upon program termination, the buffer of each process is flushed and written to the pipe.
  - so fork(2) can lead to a child process retaining data from the parent, and this explains why in such cases it may be preferable to explicitly flush any buffers before calling fork(2), just like it might be useful for a child process to call _exit(2) instead of regular exit(2) to avoid flushing of any buffers it may have inherited from its parent.
  - The exec(3) functions
    - if it has a _v_ in its name, argv's are a vector: const char *argv[]
    - if it has an _l_ in its name, argv's are a list: const char *arg0, ...
    - if it has an _e_ in its name, it takes a char *const envp[] array of environment variables
    - if it has a _p_ in its name, it uses the PATH environment variable to search for the file
