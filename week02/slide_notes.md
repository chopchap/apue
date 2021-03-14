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
