# slide notes
---
## hard disk (HDD)
- there are 3 different kinds of hard drives: `SATA`, `SSD` and `NVMe`. (Serial Advanced Technology Attachment, Solid State Drive, Non-Volatile Memory Express)
![alt text](https://github.com/chopchap/apue/blob/main/images/three%20types%20of%20HDD.png?raw=true)
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
![alt text](https://github.com/chopchap/apue/blob/main/images/naming%20schemes%20on%20NetBSD.png?raw=true)
- on NetBSD, use cmd `sudo disklabel wd0` to inspect the OS-specific partition table (a+b+?=c; c+64=d):
![alt text](https://github.com/chopchap/apue/blob/main/images/partition%20desc%20info%20on%20NetBSD%20by%20cmd%20disklabel.png?raw=true)
  - `d:` partition describing the entire disk, starting at offset 0 and ending at the last sector
  - `c:`partition describing the portion of the disk set for use by the NetBSD operating system, starting at offset 64 -- leaving the first sector available to the `BIOS` partition table
  - `b:` for use as swap space
  - `a:` the root partition where we create our entire filesystem on
- once we have created our partition table, we can then create a file system on each partition:
![alt text](https://github.com/chopchap/apue/blob/main/images/Disk%20drive%2C%20partitions%2C%20and%20a%20file%20system.png?raw=true)
  > since the entire structure of the filesystem is written in the `superblock`, to be safe, the filesystem replicates the superblock in cylinder groups, thereby allowing recovery if superblock corrupted.
- another view of filesystem data structures:
![alt text](https://github.com/chopchap/apue/blob/main/images/Another%20view%20of%20filesystem%20data%20structures.png?raw=true)
  > the name of the file is itself the hard link. (a dir entry mapping a filename to an inode is known as a "hard link")
- mapping of dir in filesystem:
![alt text](https://github.com/chopchap/apue/blob/main/images/mapping%20of%20dir%20in%20filesystem.png?raw=true)
  > - The Unix Filesystem may store the contents of a dir on reserved dir data blocks, for efficiency reasons possibly kept separate from the data blocks used for regular files
  > - '.' and '..' dir allow you to maneuver the file system hierarchy via relative paths
  > - dir name is the mapping found in a dir's parent dir
  > - now that we see why such hard links can only exist within the same filesystem, and why the `st_dev` in combination with the `st_ino` field is required to uniquely identify a file.
- 
