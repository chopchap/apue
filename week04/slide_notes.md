# slide notes
---
## hard disk (HDD)
- there are 3 different kinds of hard drives: `SATA`, `SSD` and `NVMe`. (Serial Advanced Technology Attachment, Solid State Drive, Non-Volatile Memory Express)
![three types of HDD](https://github.com/chopchap/apue/blob/main/images/three%20types%20of%20HDD.png?raw=true)
  - Pros for SATA:
    - Low cost
    - High disk sizes
  - Cons for SATA:
    - Not good for laptops
    - Requires regular de-fragmentation
  - Pros for SSD:
    - Fast
    - More durable, especially for laptops
  - Cons for SSD: 
    - More expensive than SATA drives
    - Lower disk sizes
  - Pros for NVMe: 
    - Fastest disk type on the market
  - Cons for NVMe: 
    - Extremely expensive
    - Available for desktop PCs only
    - May require replacing main board to get full benefit
- a disk can be divided into logical partitions(e.g. BIOS partition etc.)
- a `drive` is a location (medium) that is capable of storing and reading info which is not easily removed, like a disk or disc. (e.g. floppy drive, hard drive, CD-ROW drive etc.)
- `IDE` (Integrated Drive Electronics) is a standard interface for connecting a motherboard to storage devices such as hard drives and CD-ROM/DVD drives. (of course on your VM, the controller IDE is virtual)
- naming schemes on NetBSD:
![naming scheme](https://github.com/chopchap/apue/blob/main/images/naming%20schemes%20on%20NetBSD.png?raw=true)
  - with each disk slice or logical volume there are 2 methods by which they can be accessed, either through the raw (character) interface or through the block interface. (e.g. `ls -l /dev/rwd1a /dev/wd1a`)
  - when accessing the block device, data is read and written through the system buffer cache. Although the buffers that describe these data blocks are freed once used, they remain in the buffer cache until they get reused. Data accessed through the raw or character interface is not read through the buffer cache. Thus, mixing the two can result in stale data in the buffer cache, which can cause problems.
  - All filesystem commands, with the exception of the `mount` cmd, should therefore use the raw/character interface to avoid this potential caching problem.
- on NetBSD, use cmd `sudo disklabel wd0` to inspect the OS-specific partition table (a+b+?=c; c+64=d):
![partition desc info on NetBSD by cmd disklabel](https://github.com/chopchap/apue/blob/main/images/partition%20desc%20info%20on%20NetBSD%20by%20cmd%20disklabel.png?raw=true)
  - `d:` partition describing the entire disk, starting at offset 0 and ending at the last sector
  - `c:`partition describing the portion of the disk set for use by the NetBSD operating system, starting at offset 64 -- leaving the first sector available to the `BIOS` partition table
  - `b:` for use as swap space
  - `a:` the root partition where we create our entire filesystem on
- once we have created our partition table, we can then create a file system on each partition:
![disk drive partitions and filesystem](https://github.com/chopchap/apue/blob/main/images/Disk%20drive%2C%20partitions%2C%20and%20a%20file%20system.png?raw=true)
  > since the entire structure of the filesystem is written in the `superblock`, to be safe, the filesystem replicates the superblock in cylinder groups, thereby allowing recovery if superblock corrupted.
- another view of filesystem data structures:
![another view of filesystem data structures](https://github.com/chopchap/apue/blob/main/images/Another%20view%20of%20filesystem%20data%20structures.png?raw=true)
  > the name of the file is itself the hard link. (a dir entry mapping a filename to an inode is known as a "hard link")
- mapping of dir in filesystem:
![mapping of dir in filesystem](https://github.com/chopchap/apue/blob/main/images/mapping%20of%20dir%20in%20filesystem.png?raw=true)
  > - The Unix Filesystem may store the contents of a dir on reserved dir data blocks, for efficiency reasons possibly kept separate from the data blocks used for regular files
  > - '.' and '..' dir allow you to maneuver the file system hierarchy via relative paths
  > - dir name is the mapping found in a dir's parent dir
  > - now that we see why such hard links can only exist within the same filesystem, and why the `st_dev` (i.e. the device number) in combination with the `st_ino` (i.e. inode number) field is required to uniquely identify a file.
  > - on the _root_ filesystem, both '.' and '..' in the root directory _do_ point to the same inode. But that is the only instance of dot and dot dot pointing to the same directory, since only in the root do you **NOT** have a parent directory.
## link(2)
- since reasons listed, symbolic links were invented:
  1. Unix systems have not implemented hardlinks across filesystem
  2. creating a hardlink to a dir is not permitted unless the effective UID is 0. (hardlink to a directory might cause a filesystem hierarchy loop)
## unlink(2), see `wait-unlink.c`
- the system will only release the data blocks when both the link count is 0 and no process has an open file handle to this file.
## rename(2), see `rename.c`
- tmpfs is a separate system mounted on `/tmp` -- a memory filesystem in many Unix or Unix-like operating systems. (`df` can check to see)
## symlink(2), see `lns.c`
- it's possible to create cyclical chains of symlinks. The filesystem detects this and errors out rather than going into infinite dereferencing loops.
- unlike hardlinks, you can create symbolic links across filesystems
- `ls -l` can tell us what the symlink points to with **readlink(2)** syscall since there's no lopen(2)
## directories, see `simple-ls.c`
- opening dirs and listing its contents requires `r--` permissions on the dir, but accessing any files inside a dir will require `--x` on the dir
- the fts(3) library provide a lot of additional convenience if you want to traverse file system hierarchies (these lib functions actually call opendir/readdir syscalls; Not guaranteed to be available on all Unix versions)
## chdir(2): see `cd.c`
- The current working directory is specific to the current process, and changing the current working directory can only work within the same process, which is why 'cd' must always be a shell builtin and cannot be a standalone executable.  Even if your OS may ship one.
## jot(1)
- e.g. `jot 40 3`, print 40 numbers start from 3
## xargs(1)
- the xargs utility reads space, tab, newline and end-of-file delimited strings from the standard input and executes utility with the strings as arguments.
- e.g. `jot 40 3 | xargs touch` to create 40 files
## create a long file name
- `yes a` - output 'a' string forever
- `head -1024` - display first 1024 lines of a file
- `tr -d '\n'` - delete '\n' characters from the input
- `touch $( yes a | head -255 | tr -d '\n' )`: create a file name with 255 characters of 'a'
## start from scratch with a brand new filesystem on our disk
> 1. cd /
> 2. sudo umount /mnt
> 3. sudo newfs -b 4096 /dev/rwd1a
> 4. sudo mount /dev/wd1a /mnt
> 5. sudo chown chopchop /mnt
## hexdump(1)
- use the 'hexdump' utility to provide us with the
hexadecimal representation of the bytes on disk.
- on NetBSD, you can `hexdump -C .` to see a what's in a dir
- by examine this, we know that how the filesystem manages the directory: it uses the directory entry length to identify where the next directory entry begins; but if the filename length of the entry (plus padding) is shorter than the directory entry, then we have space to add a new entry at this point.
- But removing a directory entry doesn't have to go and erase the data inside of the directory. All we need to do is update the previous directory entry's length to span the entry we just removed.
- directories are filesystem dependent, and different filesystems may implement them differently. (e.g. / and /tmp are different from each other)
- a directory may grow in size, but not shrink, due to the way it juggles the different entries of varying lengths.
## bc(1)
- use 'bc' utility to easily convert numbers between different bases as shown here
- e.g. `echo "ibase=16; 4B80" | bc` to convert a 16-based 4B80 into a decimal number
## /etc/passwd - the user database
![/etc/passwd fields](https://github.com/chopchap/apue/blob/main/images/:etc:passwd%20fields.png?raw=true)
- the login shell can be set to any program. Since we have a number of service accounts that we only use for privilege separation (e.g. `sshd, ntpd ...`) -- that is, to allow a process to have a dedicated UID and not have any other privileges -- but that we do not ever want to allow to log in interactively, we can set the login shell to /sbin/nologin.
## finger(1)
- The 'finger' command can be used to find out information about a user account. (e.g. `finger root`)
## /etc/master.passwd, see `getpw.c`
- This file contains all the fields from /etc/passwd, plus the additional fields. And it also includes the hashed password, because /etc/passwd needs to be readable by all users to allow the _getpwuid(3) / getpwnam(3)_ calls to succeed without super-user privileges. (in Linux, /etc/shadow is the counterpart)
- /etc/group is much the same as /etc/passwd, but a bit complicated, see `groups.c`
- accessing similar system data files:
![accessing similar system data files](https://github.com/chopchap/apue/blob/main/images/accessing%20system%20data%20files.png?raw=true)
## mtime, atime, ctime
- mtime & atime: modification & accessing time of file data, ctime: represents a change in the struct stat itself
- `ls -l` by default print entry by mtime
- `ls -lu` .. by atime, `ls -lc` .. by ctime
- on NetBSD Unix's /dev/wd0a filesystem (mounted on /) for example, by default, any read on any file will trigger an update of the atime, which the filesystem then has to write to disk. That means that you constantly are triggering comparatively expensive I/O, which not only impacts overall I/O performance, but especially on solid state drives, this can actually reduce the lifespan of the drive. That's why many filesystem support a mount option to disable atime updates altogether - `noatime`; On Linux, the filesystem mount option is known as 'relatime', or "relative atime". the atime is only updated if the mtime or ctime is newer than the atime, or if the atime is older than 24 hours. This allows the filesystem to continue to support applications that depend on the atime change, but to not have to thrash the disk with I/O on read-only operations.
- os uses `utime(3)` to perform time manipulations on files
## gettimeofday(2), see `time.c`
- get time elapsed since the epoch
- `clock_gettime(2)` to get granularity nanoseconds
- `diff -bu file[12]`
  - `-b`: ignore changes in the amount of white space
  - `-u`: output lines of unified context
- `$ date +%s` cmd has the same effect
- `gmtime(3)` to break down the time_t into a struct `tm`, 
- `asctime(3)` takes a struct tm and formats it into a string readable for human
- `mktime(3)` operates in the inverse direction from gmtime/localtime.
- `ctime(3)` is like asctime(3) but takes a `time_t *` instead of a `struct tm *`
- `strftime(3)` can format time like `$ date +%Y-%m-%dT%H:%M:%SZ`
- picture for converting one into another:
![time is headache](https://github.com/chopchap/apue/blob/main/images/time%20is%20a%20concept.png?raw=true)
