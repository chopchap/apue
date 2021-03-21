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
  > - now that we see why such hard links can only exist within the same filesystem, and why the `st_dev` in combination with the `st_ino` field is required to uniquely identify a file.
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
