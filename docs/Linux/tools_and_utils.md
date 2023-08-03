title: Tools & Utils

# **Tools & Utils**


## **Compression**

=== "zstd"

    A directory:
    ```bash
    # Compress:
    tar -I "zstd --threads 30 --ultra -19 --memory 8192" -cf scripts.tar.zst scripts/
    
    # Decompress:
    tar --zstd -xf scripts.tar.zst
    ```

    A single file:

    ```bash
    # Compress:
    zstd --compress --threads 30 --ultra -19 --memory 8192 -o file.txt.zstd file.txt

    # Decompress:
    zstd --decompress file.txt.zstd
    ```

A couple of observations:

* zstd uses cores up to the compression level of 19 (including).
* `--ultra -22` gives the best compression


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






## **Hash of a directory**

```bash
find ./ -type f -exec sha256sum {} + | sha256sum
```


## **Mouse scroll speed**

See [askubuntu](https://askubuntu.com/questions/285689/increase-mouse-wheel-scroll-speed/621140#621140).
