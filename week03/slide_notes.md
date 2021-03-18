# slide notes
---
## stat(2)
- to obtain information about a file (i.e. to get all the _meta_ information, not the contents of the file)
- `lstat(2)` get the information about the symlink
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

## stat(1) -r file_name
- Display raw information in struct stat
- `st_dev`: identifies the device ID the file resides on, i.e., the disk
- `st_ino`: identifies a file's inode number across all mount points in the file system
- `st_mode`: encodes the file type as well as the permission
- `st_nlink`: the link count
- `st_uid & st_gid`: the numeric UID and the group ID
- `st_atime, st_mtime & st_ctime`: the file's last access, modification, and file status change times respectively
- a dir is a file that maps symbolic names to inode numbers, allowing the FS to find the file meta info and data blocks associated. Such a mapping between an
indoe and a string is known as a "link" or a "hard link"
- a device file under /dev dir comes as either character special devices(e.g. terminals), and block devices(e.g. disks)
- a fifo(named pipe) is an interprocess communication endpoint to allow unrelated processes to communicate with one another
- symbolic links are files that contain only content the pathname of another file
## ls -li file file2
- shows us the inode numbers of the files
## ls -ls file
- Display the number of file system blocks actually used by each file, in units of 512 bytes or BLOCKSIZE
- `BLOCKSIZE=4096 df .` to display the info in those sized chunks
## process IDs
- for each process, there're six or more IDs associated with it:
  ![alt text](https://github.com/chopchap/apue/blob/main/images/uid%26gid%20associated%20with%20each%20process.jpeg?raw=true)
- in some cases it is useful to allow an executable to be run with changed, often elevated privileges. In that case, we set a special bit on the file's st_mode -- the setuid bit -- and the system will set the effective UID, to that of the owner of the executable. And the real UID will remain that of the user(the same can be done using group ownership and the setgid bit)
- important take-aways:
  1) exec sets the saved-setuid and saved-setgid;
  2) changing the effective uid or effective group id is allowed to either the real uid or the saved-setuid or group ids respectively.
- `cp /sbin/ping .`, copy ping(1) cmd into current dir, and we can't execute it, since the st_mode differs from the original ping by `-r-sr-xr-x` replaced by `-r-xr-xr-x`, but use `chmod u+s ./ping` does no good, the reason is that the setuid bit is only meaningful when the st_uid field in the struct stat of the executable is different from the euid of the user executing the command
- `w` cmd presents who users are and what they are doing
- `wall` cmd write a message to users, and when 2 users logged in on the system gets a pseudo terminal(pts/0 or pts/1), when you `ls -l /dev/pts`, notice that these devices do have group write permissions set: `crw--w----`, so when this command is executed -- no matter by whom --, it will run with an effective gid of 'tty', and since group 'tty' can write to the terminals of the logged in users, the wall(1) cmd with permission `-r-xr-sr-x` succeeds.
- `setuid(2)` will succeed if the argument given is either the current real UID, or the saved-setuid, and it will _only_ set the eUID. `setuid(2)` will set _both_ the real UID, the eUID, and the saved-setuid. This means that after a call to setuid(2), you can no longer regain any possible elevated privileges you may have had before
- `access(2)`: we also may encounter situations where we are running with elevated privileges, but need to determine whether or not our real UID would be allowed to access a resource, instead of using cumbersome seteuid(2), use _access(2)_ instead, it will tell us whether the _real_ UID would be able to perform the action
- All in all, if the setuid (setgid) bit is set in the permissions, the effective UID (GID) will become that of the st_uid (st_gid) of the file's struct stat at execution time.
