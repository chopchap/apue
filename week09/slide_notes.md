# slide notes
---
## socketpair(2)
- use pipes to bi-directional communication is cumbersome, so instead we use a `socketpair(2)`
- the sockets are created in the specified _domain_, of the specified _type_, and using the specified _protocol_. (see `socketpair.c`)
  - a _domain_ is a specific address format that implies certain conventions, such as what types of communications are supported and what protocols may be spoken.
  - socketpairs are only implemented in the "UNIX" or "local" domain, using the `AF_UNIX` or `PF_LOCAL` identifier.
## socket(2)s in the LOCAL domain
- sockets API allows you to implement interprocess communications logic that is largely identical for processes communicating on the same system as for those communicating across the network. (see `udgramsend.c` & `udgramread.c`)
- in order to be able to transmit data reliably -- that is, using a stream -- a socket must be in connected state; but if using datagrams instead, we can use sendto(2) to submit data without calling `connect(2)`.
## DGRAM Sockets in the INET Domain
- communications between two hosts over the internet.
- see `dgramread.c` and `dgramsend.c`
- `$ netstat -na` shows us all the open ports on the system
- `$ tcpdump -X -n host _hostname_`: dump traffic on a network
  - `-X`: When parsing and printing, in addition to printing the headers of each packet, print the data of each packet
  - `-n`: Don't convert addresses to names
- `$ printf "%d\n" 0xffd2`: formats and prints its arguments
- `$ man ascii`: octal, hexadecimal, decimal and binary ASCII character sets
- Sockets: Datagrams in the Internet Domain
  - Unlike UNIX domain names, Internet socket names are not entered into the file system and, therefore, they do not have to be unlinked after the socket has been closed
  - The local machine address for a socket can be any valid network address of the machine, or it can be the wildcard value INADDR_ANY
  - request any ephemeral port by calling `bind(2)` with a port number of 0
  - "well-know" ports (range 1 - 1023) can only be bound by euid 0
  - determine used port number (or other information) using `getsockname(2)`
  - convert between network byte order and host byte order using `htons(3)` and `ntohs(3)`(which may be noops)
  - UDP is connectionless/unreliable: you can (try to) send packets without anything listening
## STREAM sockets in the INET6 Domain
- use TCP to establish a sequenced, reliable, two-way byte stream.
- see `streamread.c` and `streamwrite.c`
- `$ telnet _hostname_ _port_`: communicate with another host using the TELNET protocol.
- Sockets: Streams in the Internet6 Domain:
  - connections are asymmetrical: one process requests a connection, the other process accepts the request
  - one socket is created for each accepted request
  - mark socket as willing to accept connections using `listen(2)`
  - pending connections are then `accept(2)`ed
  - `accept(2)` will block if no connections are available
  - each connection requires a full handshake
## I/O Multiplexing
- `select(2)` tells us both the total count of file descriptors that are ready as well as which ones are ready
- see `two-sockets-select.c`
- file descriptor sets are manipulated using the FD* functions/macros
- readfds/writefds sets indicate readiness for read/write; exceptfds indicates an exception condition (for example OOB data, certain terminal events)
- EOF means ready for read - read(2) will just return 0 (as usual)
- `pselect(2)` provides finer-grained timeout control; allows you to specify a signal mask (original signal mask is restored upon return)
- I/O Multiplexing:
  - use `select(2)` to check multiple file descriptors for I/O readiness
  - avoid blocking individual file descriptors by handling I/O in a separate process or thread
  - alternative or similar interfaces:
    - poll(2)
    - epoll(2) (Linux)
    - kqueue(2) (*BSD)
