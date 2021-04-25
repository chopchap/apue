# slide notes
---
## POSIX ACLs
- ACLs are stored as Extended Attributes and require support of the file system
- Ordering of ACLs may be relevant -- experiment with different users and groups to verify the impact of the order.
- Different OS implement POSIX ACLs differently
  - Linux: `getfacl(1)/setfacl(1)/acl(5)`
  - macOS: `chmod(1)` to create/manipulate, `ls(1)` to inspect
  - NetBSD: upcoming in NetBSD 10.0
## eUIDs, file flags, mount options, securelevels
- `su(1)` and `sudo(8)` can be used to grant others the ability to run commands as another user, but it can be difficult to restrict access
- "file flags" may restrict certain use; see `chflags(1)/chflags(2)` on BSD, `chattr(1)` on Linux
- mount options like _noexec_, _nosuid_, _rdonly_ can restrict and protect filesystems per mount point
- to prevent even root from undoing these protections, use securelevels (reboots are noisy.)
## Restricted shells, Chroots, Jails
- Restricted shells run fully within the OS, with restrictions entirely enforced within the shell.
- A `chroot(2)` can create a severely restricted environment with a "changed root" filesystem:
  - present in most Unix versions since the 80s, but since removed from POSIX
  - chroot escapes may be possible
  - process space outside of the chroot remains visible
- Jails were introduced in FreeBSD ~2000 as the first real versoin of OS-level virtualization and predecessor of true contaniers.
## Process Priorities
- processes can voluntarily self-restrict their resource utilization
- CPU usage priority can be adjusted using `setpriority(2)`
- use `nice(1)` or `renice(8)` to adjust the niceness of your process or process group
- once you're nice, you cannot go back
- priority does not influence CPU placement
## Processor Affinity and CPU Sets
- Pinning a process (group) to a CPU can improve performance by _e.g._, reducing CPU cache misses.
- _"Processor Affinity"_ or _"CPU pinning"_ lets you assign a process to a specific CPU, but other processes may still be placed on that CPU.
- Processor Affinity is inherited by a child process from its parent, but changing a parents affinity does not affect running children.
- _"CPU Sets"_ let you reserve CPUs for specific processes; no other processes can be placed on those CPUs.
- Processor Affinity and CPU sets are not standardized; different OS implement them differently or using different tools.
## Capabilities, Control Groups, Containers
- 
