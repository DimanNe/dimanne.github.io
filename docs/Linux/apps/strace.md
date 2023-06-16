title: strace

# **strace**

Start with: `sudo strace -fp (pgrep -u (id -u) -x process_name_here) -s 1024`

## **FD's**

* **Process -> FD**: Given a process you can list all its FDs: `sudo lsof -p (pgrep -u (id -u) -x process_name_here)`.
* **FD -> inode**: Once you have FD, there is inode in the `NODE` column.
* **inode -> remote end-point**: Given the inode, you can run lsof again and check what other processes has the same inode.

    Alternatively, you can use `find`: `sudo find / -inum 117473778 2>/dev/null` to find a file by inode.

## Forks

Track only creation of new processes: `strace -ftts 4096 -e trace=fork,vfork,clone,execve ytest`

