# slide notes
---
## getconf(1) & sysconf(3)
- `getconf system_var`, getconf(1) references sysconf(3)
- sysconf(3) gets us a _configurable_ system variable, such like `_SC_OPEN_MAX`
- the return value of sysconf(3) is meaningful; we get back -1 on error, with errno set appropiately, but we also need a way to identify when we can't find the variable we were asked to look up, but we can't return '0', since that might be a valid value. So we return -1 again, but leave errno unmodified (hence the code in `openmax.c`).
## which cmd
- by typing `which cmd(1)`, you can locate a program file in the users $PATH environment variable, then you can find the source code location of this cmd(1), e.g. /usr/bin/getconf is environment variable of `which getconf`, then you can find it's source code in /usr/src/usr.bin/getconf/getconf.c
## ulimit -n 64
- we can inspect what the current limit in our shell is via the 'ulimit' builtin by typing `ulimit -n`. (This illustrates that the maximum number of open files per process is configurable at runtime, which is why the sysconf(3) function allows you to retrieve it equally at runtime instead of looking up a fixed constant from a header file)
## uname -v
- print the operating system version
## creat(2)
- create(2) returns a file descriptor opened in write-only mode, and in the early days of Unix, when open(2) wasn't able to create files, you had to creat(2) the file, close(2) it, then open(2) it again, which isn't an _atomic_ operation. So this call is now obsoleted by by open(2) with the flags used as:  `open(path, O_CREAT|O_TRUNC|O_WRONLY, mode)`.
## open(2)
- int open(const char *path, int oflags, ...);
- oflag must be one (and only one) of:
  - _O_RDONLY_
  - _O_WRONLY_
  - _O_RDWR_
- oflag may be OR'd with:
  - _O_CREAT_: (if oflag O_CREAT is specified, then the third argument 'mode' is required. If file already exists, it also will succeed if without O_EXCL OR'd)
  - _O_EXCL_: (Error if O_CREAT and the file already exists)
  - _O_NONBLOCK_ ...
- the third parameter mode is optional, and subject to modifications by the process's umask(2)
- the only syscall that doesn't take a file descriptor.
- openat(2) works the same way as open() except if `path` is relative, it is looked up from a directory whose file descriptor was passed as `dirfd`.
## close(2)
- closing a fd will release any _record locks_ on that file. close(2) can fail, but we can explicitly cast it to void to avoid compiler complains.
