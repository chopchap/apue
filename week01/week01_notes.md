# week01_notes
---
### **Single UNIX Specification (POSIX)**
   > *POSIX.1-2017 defines a standard operating system interface and environment, including a command interpreter (or “shell”), and common utility programs to support applications portability at the source code level.*

### **Environment Setting up**
   1. download & install VirtualBox by default settings so that you can use `VBox*` series command stand by.
   2. use this link: https://cdn.netbsd.org/pub/NetBSD/NetBSD-9.0/images/NetBSD-9.0-amd64.iso to download NetBSD ISO file.
   > *(what is an **ISO image**? The name ISO was taken from the name of the file system used by optical media, which is usually ISO 9660. You can think of an ISO image as a complete copy of everything stored on a physical optical disc like CD, DVD, or Blu-ray disc—including the file system itself. They are a sector-by-sector copy of the disc, and no compression is used. The idea behind ISO images is that you can archive an exact digital copy of a disc, and then later use that image to burn a new disc that’s in turn an exact copy of the original. Most operating systems (and many utilities) also allow you to mount an ISO image as a virtual disc, in which case all your apps treat it as if a real optical disc were inserted.)*
   3. specific operations: https://stevens.netmeister.org/631/virtualbox/
   4. we initially boot off the CD, but then boot off the disk once the OS installed without having to eject the CD from the VM(i.e. set Hard Disk as the first boot order option in storage setting in VBox)
   5. VDI or Virtual Drive Format is used for saving the Virtual Drive Image(**.vdi**) file.
   6. network setting: select first adapter, keep it as **NAT**
   > *(Network Address Translation is designed for IP address conservation. It enables private IP networks that use unregistered IP addresses to connect to the Internet. NAT operates on a router, usually connecting two networks together, and translates the private (not globally unique) addresses in the internal network into legal addresses, before packets are forwarded to another network.)*
   ![alt text](https://github.com/chopchap/apue/blob/main/images/Port%20forwarding%20via%20NAT%20router.png?raw=true)
   7. `MBR`:
   > *The Master Boot Record is the information in the first sector of any hard disk or diskette that identifies how and where an operating system is located so that it can be boot (loaded) into the computer's main storage or random access memory. The Master Boot Record is also sometimes called the "partition sector" or the "master partition table" because it includes a table that locates each partition that the hard disk has been formatted into. In addition to this table, the MBR also includes a program that reads the boot sector record of the partition containing the operating system to be booted into RAM. In turn, that record contains a program that loads the rest of the operating system into RAM.*
   8. `BIOS`: 
   > *(basic input/output system) is the program a computer's microprocessor uses to start the computer system after it is powered on.*
   9. `pkgin`: a binary package manager for pkgsrc. 
   > *Many so-called GNU/Linux distributions provide a convenient way of searching, installing and upgrading software by using binary archives found on "repositories". NetBSD, and more widely, all operating systems relying on pkgsrc have tools like pkg_add and pkg_delete, but those are unable to correctly handle binary upgrades, and sometimes even installation itself.*
   10. `ssh`(secure shell):
   > *ssh is a network protocol that gives users, particularly system administrators, a secure way to access a computer over an unsecured network*
   11. `ntp`(Network Time Protocal): 
   > *When NTP is enabled, your device contacts an NTP server to synchronize the time. When NTP is enabled, you can optionally enable your Firebox as an NTP server. When you enable your device as an NTP server, clients on your private networks can contact your Firebox to synchronize the time.*
   12. `mount` for mounting file system: 
   > *whilst we are in the process of configuration, when all set, before reboot, mount(“mounting” a disk volume on a server into a virtual “local disk drive”, so programs can access it just like a regular physical local disk drive.) the virtual disk and edit the file /etc/rc.conf on it by typing:*
      > - `mount /dev/wd0a /mnt`
      > - `vi /mnt/etc/rc.conf`
   13. Naming scheme: 
   > *usually, `/dev/wd0` is the name of the first disk and `/dev/wd1a` the name of the first partition on second hard disk in NetBSD (whereas /`dev/hda` and `/dev/hdb1` in Linux)*
   14. Set up ssh
   > *since we've set port 2222 as the communicating pipe between host and guest os, we can use command `$ ssh -p 2222 usr_name@127.0.0.1` to access guest os. Now, on the first handshake, you may need to set fingerprint, which means on your host os, use command `$ ssh-keygen -b 4096 -f ~/.ssh/filename`, this will effectively generate a record in file ~/.ssh/know_hosts and files filename & filename_pub in dir ~/.ssh/, then we need to copy the public key to your VM and install it under ~/.ssh/authorized_keys by using command `$ scp -P 2222 ~/.ssh/filename.pub usr_name@127.0.0.1:` (or you use `$ sftp usr_name@xx.xx.xx.xx` to put it in the dir of your remote guest lab os), then on your guest os, `$ mv filename.pub ~/.ssh/authorized_keys`, now you are able ssh into guest os by using `$ ssh -p 2222 -i ~/.ssh/filename usr_name@127.0.0.1`. Last step, to be able to be simply typing `$ ssh guestos` to log in, we write a heredoc:*
   ```txt
   cat >> ~/.ssh/config << EOF
   heredoc> Host guestos
   heredoc> HostName xx.xx.xx.xx
   heredoc> IdentityFile ~/.ssh/filename
   heredoc> Port 2222
   heredoc> User usr_name
   heredoc> EOF
   ```
