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
## read(2)
- _your_ responsibility to ensure that the buffer you hand to read(2) is big enough to read num bytes into.
- if you ask read(2) to read 256 bytes of data, but there are only 32 bytes of data left, then it will read those 32 bytes; a subsequent read(2) call will then return 0 to indicate EOF
- there may not be as much data available as you have requested, even though we have not yet reached EOF. For example, if you're reading from a socket, it's possible that not enough data has arrived yet, so read(2) might return to you however many bytes are available, but upon a subsequent call, there may be new data available.
## write(2)
- In general, write(2) will block until is has completed writing the requested number of bytes, but if in _`non-blocking`_ mode, and if the object represented by the file descriptor is subject to flow control (such a network sockets, for example), then write(2) my write fewer than the requested number, and you'd have to retry writing the remaining bytes yourself.
- for regular files, write begins writing at the current offset (unless O_APPEND has been specified, in which case the offset is first set to the end of file)
- if the real user is not the root, then write(2) clears the set-user-id bit on a file. This prevents the penetration of system security by a user who "captures" a writable set-user-id file owned by the root
## lseek(2)
- a pipe is *`record oriented`*, meaning you can't move back and forth on the data as it comes in.  You can't seek on a pipe.
## df -h .
- display free disk space
## ls -ls file_name
- Display the number of file system blocks actually used by each file, in units of 512 bytes or BLOCKSIZE
## hexdump -c file_name
- Display the input offset in hexadecimal, followed by sixteen space-separated, three column, space-filled, characters of input data per line.
- when a sparse file(or a hole file) being copied, some system might fill in NULL bytes in the hole, others don't, it depends on the file system
## diff file1 file2
- diff - compare files line by line
## stat -f "%k" tmp/file1
- a way to determine the optimal I/O size, i.e., the filesystem blocksize, of a given file
- the command-line syntax for the stat(1) command differs between Unix versions, check manual (e.g. `stat -c "%o"` in Linux)
## kernel data structures for open files
> ![alt text](https://github.com/chopchap/apue/blob/main/images/Kernel%20data%20structures%20for%20open%20files.jpeg?raw=true)
- _`vnode`_ allows for an abstraction of the filesystem API. Different file systems include different meta information in the vnode table entry, but for compatibility with the traditional Unix filesystem, they implement or provide all the inode information
- If a process opens the same file twice, then the file table entries are distinct, but pointing to the same vnode table entry (Things are the same if two different processes opened the same file)
- file status flag _`O_APPEND`_ is _atomic_, so that we don't need to do 2 ops: lseek(2) first an then write(2), which rises race conditon
- sometimes we want to write data to a different location besides the end and avoid the race condition. pread(2) and pwrite(2), take an additional `offset` parameter and will _atomically_ seek before issuing the read(2) or write(2) operation.
## '>' operator in shell
- in practical shell environment, it uses these system calls to create files
  - `$ echo foo > file # O_WRONLY | O_CREAT` if file doesn't exist
  - `$ echo bar > file # O_WRONLY | O_CREAT | O_TRUNC` if file already exists
  - `$ echo foo >> file # O_WRONLY | O_CREAT | O_APPEND`
- `ls -l file /nowhere 2>/dev/null`, using the special device /dev/null we are able to ditch the output from stderr
- in general, messages to stderr are _unbuffered_,
while I/O to stdout is likely to be _buffered_, which is why you will likely see error messages prior to normal output when both are connected to a terminal, use syntax `2>&1` instead if you want to tell the shell to put stderr wherever stdout is regardless of where stdout goes to(e.g. `ls -l file /nowhere 2>&1 | nl`, i.e. `dup2(STDOUT_FILENO, STDERR_FILENO)` in shell source code)
## dup(2)
- The dup(2) system call duplicates an existing file descriptor, and you get back a second file handle pointing to the same file table entry as the first one, meaning the two share the offset and file status flags etc. *(unlike in the case where a process has called 'open(2)' on the same files multiple times and gets back file desriptors pointing to distinct file table entries that then point to the same vnode table entry)*
- for redirection of an existing newfd since newfd in dup2(oldfd, newfd), it'll close(newfd) first
## fcntl(2)
- fcntl(2) comes into play if we want to inspect or change file status flag when we didn't call open(2) ourselves(i.e. when all we have is a fd)
- synchronous output will wait for I/O to be flushed to disk after each call, while asynchronous output allows the OS to handle this more efficiently and cache I/O a bit
