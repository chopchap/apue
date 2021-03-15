# Slide notes
---
## Codes
### simple-shell.c
- **fgets:** *get a line from a stream, EOF(ctrl+D) to quit*
- **errno** *will be set if some error occurs, and it'll be a number, so better use strerror in additon to offer clarity*
### simple-ls.c
- **opendir & readdir:** *to list contents in a dir, open it first, then readdir, return types are `DIR *` and `struct dirent *` respectively*
- **closedir**: *don't forget to close*
---
## Commands
- `find /usr/src/usr.bin -name '*.[ch]'` *means find files like xx.c or xx.h under dir /dir/to/file*
- `time find /usr/src/usr.bin -name '*.[ch]'` *means output clock/user CPU/sys CPU time for excuting command (user CPU + sys CPU != clock since I/O processing)*
- `-exec cat {} \;` *invoke arbitrary commands with -exec action, like `-exec cmd {} ;` where `{}` is a symbolic representation of the current pathname, `;` is a required dilimiter indicating the end of the cmd. since the brace and simicolon characters have special meaning to the shell, you might need to quote or escape them with `\;` or `';'`*
- `wc -l` *list the total number of lines in file*
- `time find /usr/src/usr.bin -name '*.[ch]' -exec cat {} \; | wc -l` *means time a series of cmd(1. find specific files in specific dir; 2. use the outcome of find as parameter to excute cat cmd; 3. print total number of line) and output*
- `time find /usr/src/usr.bin -name '*.[ch]' -print | xargs cat | wc -l` *can use cat a few less times and thus boosting the performace*
---
## Extras
- **Standard I/O**, *kernel provides **unbuffered** I/O through e.g. open(2), read(2), write(2), lseek(2), close(2) and **buffered** I/O through e.g. fopen(3), fread(3), fwrite(3), getc(3), putc(3)*
- ![alt text](https://github.com/chopchap/apue/blob/main/images/Architecture%20of%20Unix%20operating%20system.jpeg?raw=true)
