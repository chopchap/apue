# slide notes
---
## stat(2)
- to obtain information about a file (i.e. to get all the _meta_ information, not the contents of the file)
- the `fstat(2)` version differs from the `stat(2)` in that it accepts a fd
- the `fstatat(2)` syscall allows for atomically accessing a relative pathname outside of the current working directory (which is similar to `openat(2)`)
- `struct stat`: the idiomatic way for a function to efficiently provide complex information as well as follow the convention of a simple, meaningful error code is to take as input _a pointer to a data structure_ that the call then fills in.
- on your MacOS (you can use GUI to do the same), command-line to effectively plugged in a new hard
drive into our server: 
  1. use `VBoxManage createmedium disk --format VMDK --size 50 --filename ~/VirtualBox\ VMs/apue/disk2.vmdk` cmd to create a small second disk to better understand the file system. (VBoxManage is a cmd that you can invoke once you've already download VirtualBox, VMDK is a kind of virtual memory disk, and this file will be put into a relative dir)
  2. use `VBoxManage storageattach apue --type hdd --storagectl IDE --port 0 --device 1 --medium ~/VirtualBox\ VMs/apue/disk2.vmdk` cmd to attach the new disk to our VM as an IDE drive.
- then, once we have VM booted and log in, on your host server, use:
    1. `sudo newfs -b 4096 /dev/rwd1a` cmd to create a new filesystem on the disk.
    2. next, `sudo mount /dev/wd1a /mnt` to mount the new disk under /mnt
    3. then, `sudo chown chopchop /mnt` to change the ownership to our user, so we don't need superuser privileges to write data to the new disk
    4. `cd /mnt & df .` to take a look at the available space
    5. `dd if=/dev/zero of=file bs=1024 count=1024` to create a simple, 1 MB file
    6. if you `ls -l file`, you can see each of the fields in the output is derived from the information in the `struct stat`
