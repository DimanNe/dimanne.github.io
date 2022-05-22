title: Tools & Utils

# **Tools & Utils**


## **Compression**

=== "zstd"

    A directory:
    ```bash
    # Compress:
    tar -I "zstd -T16 --ultra -22" -cf scripts.tar.zst scripts/
    
    # Decompress:
    tar --zstd -xf scripts.tar.zst
    ```

    A single file:
    ```bash
    # Compress:
    zstd --compress --threads 16 --ultra -22 -o file.txt.zstd file.txt

    # Decompress:
    zstd --decompress file.txt.zstd
    ```



## **ip commands**

10 Useful IP commands:

* `ip addr show` --- show all IP Addresses of all interfaces.
* `ip addr add 192.168.50.5 dev eth1` --- assign a IP Address to a specific interface.
* `ip addr del 192.168.50.5/24 dev eth1` --- remove an IP Address.
* `ip link set eth1 up` --- enable a network interface.
* `ip link set eth1 down` --- disable a network interface.
* `ip route show` --- show route table.
* `ip route add 10.10.20.0/24 via 192.168.50.100 dev eth0` --- add a static route.
* `ip route del 10.10.20.0/24` --- remove a static route.
* `ip route add default via 192.168.50.100` --- add default gateway.




## **Alternatives**

Add clang alternatives:

```bash linenums="1"
sudo update-alternatives                                             \
      --install /usr/bin/clang   clang   /usr/bin/clang-3.8     50   \
      --slave   /usr/bin/clang++ clang++ /usr/bin/clang++-3.8        \
      --slave   /usr/bin/lldb    lldb    /usr/bin/lldb-3.8           \
      --slave   /usr/bin/llvm-symbolizer llvm-symbolizer /usr/bin/llvm-symbolizer-3.8
```

Display info about given alternative:

```bash linenums="1"
update-alternatives --display clang
clang - auto mode
  link currently points to /usr/bin/clang-3.5
/usr/bin/clang-3.5 - priority 50
  slave clang++: /usr/bin/clang++-3.5
  slave lldb: /usr/bin/lldb-3.5
Current 'best' version is '/usr/bin/clang-3.5'.
```



## **OpenSSL**

### Certificates

Print info about a certificate in...

* p7b PEM: `openssl pkcs7 -print_certs -inform PEM -in ~/CertificationAuthority.crt`
* p7b DER: `openssl pkcs7 -print_certs -inform DER -in ~/CertificationAuthority.crt`
* x509 PEM: `openssl x509 -inform pem -text -noout -in ~/CertificationAuthority.crt`
* x509 DER: `openssl x509 -inform der -text -noout -in ~/CertificationAuthority.crt`

### Create bitcoin keys

See this [stackexchange](https://bitcoin.stackexchange.com/questions/59644/how-do-these-openssl-commands-create-a-bitcoin-private-key-from-a-ecdsa-keypair).




## **Permissions**

Restore default permissions (`x` for dirs, no `x` for files):

=== "chmod with `X`"

    ```bash
    chmod -R u=rwX,g=rwX,o=rwX ${HOME}
    ```
    
    `X` permission: execute/search only if the file is a directory or already has execute permission for some user.

=== "find + chmod"

    ```bash linenums="1"
    find ${HOME} -type f -exec chmod 0664 {} +
    find ${HOME} -type d -exec chmod 0775 {} +
    ```
    
    Note, unlike `X`, it will remove `x` permission from files.
   
=== "ACL"

    ```bash
    setfacl -dm u::rws,g::rwx,o::rx /dir/
    ```



## **initrd.img**

* Extract `initrd.img`: `unmkinitramfs /boot/initrd.img-5.15.0-30-generic ../new-init/`




## **udev**
Docs: [debian.org/udev](https://wiki.debian.org/udev), 
[freedesktop.org multiseat](https://www.freedesktop.org/wiki/Software/systemd/multiseat/),
[intro](https://opensource.com/article/18/11/udev).

### Example

Example of running a script on new flash-drive insertion.

Add the following file:


??? info inline end "Paths convention"

    The rules files (which amount to more configuration for udevd) are taken from `/run/udev/rules.d`,
    `/etc/udev/rules.d` or `/lib/udev/rules.d`. Packages install rules in `/lib/udev/rules.d`), while the
    `/etc` and `/run` locations provide a facility for the administrator to override the behaviour of a
    package-provided rule.

    If a file with the same name is present in more than one of these directories then the latter(s) file
    will be ignored. Files in there are parsed in alpha order, as long as the name ends with `.rules`.


```bash linenums="1" title="/etc/udev/rules.d/80-local.rules"
SUBSYSTEM=="block", ACTION=="add", RUN+="/usr/local/bin/trigger.sh"
```

Reload rules: `udevadm control --reload`

Refining the rule into something useful:

* `lsusb: Bus 003 Device 005: ID 03f0:3307 TyCoon Corp.` In this example, the `03f0:3307` before 
  `TyCoon Corp.` denotes the `idVendor` and `idProduct` attributes. You can also see these numbers
  in the output of `udevadm info -a -n /dev/sdb | grep vendor`, but I find the output of lsusb a
  little easier on the eyes.
* You can now include these attributes in your rule:
    ```bash linenums="1" title="/etc/udev/rules.d/80-local.rules"
    SUBSYSTEM=="block", ATTRS{idVendor}=="03f0", ACTION=="add", RUN+="/usr/local/bin/thumb.sh"
    ```

### Debugging
* While running: `udevadm monitor --property`
* Insert your device, and find string like `DEVNAME=/dev/bus/usb/003/006` in the output. 
  `/dev/bus/usb/003/006` is your device name.
* Once you have a device path, you can:
    * Get all its attributes: `#!bash udevadm info -a -n /dev/bus/usb/003/006`
    * Or simulate rule execution: `udevadm test (udevadm info -q path -n /dev/bus/usb/003/006)`


### Example of removing tags uaccess and seat tags
```bash linenums="1" title="/etc/udev/rules.d/99-multiseat-fixes.rules"
SUBSYSTEM=="video4linux", ACTION=="add", RUN="/usr/local/bin/trigger.sh"
SUBSYSTEM=="video4linux", ACTION=="add", TAG-="uaccess"
SUBSYSTEM=="video4linux", ACTION=="add", TAG-="seat"
```

Unfortunately, it means that the file associated with the device will have root as its owner, and
therefore it will not be accessible by neither of the users. Solution is to add both users to video group:

```bash linenums="1" title="/usr/local/bin/trigger.sh"
#!/usr/bin/bash 
/usr/bin/date >> /tmp/udev.log
```



## **screen**

### Enable scroll

```linenums="1" title="~/.screenrc"
defscrollback 10000
termcapinfo xterm* ti@:te@
```

### Rename existing screen

```
Ctrl + a
:sessionname mySessionName<ENTER>
```



## **Hash of a directory**

```bash
find ./ -type f -exec sha256sum {} + | sha256sum
```


## **Mouse scroll speed**

See [askubuntu](https://askubuntu.com/questions/285689/increase-mouse-wheel-scroll-speed/621140#621140).
