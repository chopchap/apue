# slide notes
---
## the login process
- boot sequence:
  - once the kernel has bootstrapped the system, it will hand off control to `init(8)`, and any subsequent messages sent to the serial console are not preserved in file `/var/run/dmesg.boot` anymore.
  - The system may be configured to allow connections on multiple terminals via the file /etc/ttys, so for each entry there, `init(8)` will `fork(2)` a new process.
  - and `exec(3)` `getty(8)`, the program responsible for configuring the tty, read the login name, and then invoke the `login(1)` command.
  - When a user enters their username on the terminal, `getty(8)` will `exec(3)` the `login(1)` program.
  - Upon successful login, the `login(1)` program then `exec(3)`s the user's shell
  - for every command entered in the shell, the shell then `fork(2)`s a new process, then `exec(3)` the given command.
  - When the shell exits (i.e. exit or ctrl+D), then `init(8)` will reap the process, fork(2) again, and exec `getty(8)` on the now available terminal again.
  - The first few steps of the login process all run with superuser privileges, but then `login(1)` comes in. It will turn off echo on the terminal so your password is not shown when you type it; then it reads your password, hashes it, and compares the hash to what's found in /etc/master.passwd.
  ![unix login](https://github.com/chopchap/apue/blob/main/images/Unix%20login.png?raw=true)
## process groups and sessions
- `getpgrp(2)` & `getpgid(2)`
  - group leader's pid == gid
- sessions:
  - ![sessions](https://github.com/chopchap/apue/blob/main/images/sessions.png?raw=true)
  - the different process groups in the session may interact with the controlling terminal in different ways: the foreground process group may receive keyboard input as well as keyboard generated signals, and a hangup of the terminal will send a signal to the session leader (which is shell's pid in this case).
  - ![process groups](https://github.com/chopchap/apue/blob/main/images/process%20groups.png?raw=true)
## job control
- session control:
  ![session control](https://github.com/chopchap/apue/blob/main/images/job%20control.png?raw=true)
## signals
- `kill(2)` & `signal(3)`, see `sigusr.c`
  - to send a signal and install a signal handler
  - you also have the option to say "not now" by blocking the signal from being delivered.  This is different from ignoring the signal in that you can then unblock the signal at a later point and then see whether any such signal did get delivered and then have it take either of the above three actions when you're ready.
- `signal1.c`: we can be interrupted within a signal handler as well as that multiple blocked signals are merged into one and then delivered when unblocked.
- `sigprocmask(2)`, see `signal3.c`
## Reentrant & interrupted functions
- there are even more severe consequences of making calls that are not async-signal-safe. (see `reentrant.c`)
- `alarm(1)` get program interrupted every second
- Some functions are either non-interruptable, guaranteed to be _atomic_. Or the function is known to be _reentrant_, that is, multiple invocations may safely run concurrently.
- Blocking functions (see `eintr.c`): if a signal handler is invoked while a system call or library function call is blocked, then:
  - the call may be forced into terminate with the error EINTR
  - the call may return with a data tranfer shorter than requested
  - the call is automatically restarted after the signal handler returns
