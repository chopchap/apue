# slide notes
---
## the login process
- executing order:
  - once the kernel has bootstrapped the system, it will hand off control to init(8), and any subsequent messages sent to the serial console are not preserved in file /var/run/dmesg.boot anymore.
  - The system may be configured to allow connections on multiple terminals via the file /etc/ttys, so for each entry there, init(8) will fork(2) a new process.
  - and exec(3) getty(8), the program responsible for configuring the tty, read the login name, and then invoke the login(1) command.
  - When a user enters their username on the terminal, getty(8) will exec(3) the login(1) program.
  - Upon successful login, the login(1) program then exec(3)s the user's shell
  - for every command entered in the shell, the shell then fork(2)s a new process, then exec(3) the given command.
  - When the shell exits (i.e. exit or ctrl+D), then init(8) will reap the process, fork(2) again, and exec getty(8) on the now available terminal again.
  - The first few steps of the login process all run with superuser privileges, but then login(1) comes in. It will turn off echo on the terminal so your password is not shown when you type it; then it reads your password, hashes it, and compares the hash to what's found in /etc/master.passwd.
  ![unix login](https://github.com/chopchap/apue/blob/main/images/Unix%20login.png?raw=true)
- 
