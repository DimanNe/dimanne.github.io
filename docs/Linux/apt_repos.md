title: apt repos

# **apt repos**

### **List installed** packages

```bash
apt list --installed
```



### Find **package by name**: `apt search "qt.*5.*-dev"`

For example find all `qt5 dev` packages needed for development:
```bash
apt search "qt.*5.*-dev" | grep -v "i386" | grep -v "py"
```

or find all corresponding packages with debug symbols:
```bash
apt search "qt.*5.*-dbg" | grep -v "i386"
```


### Find **package by name**: `apt search --names-only`

Another alternative, to search only in package names:
```bash
apt search --names-only python
apt list | grep -i bazel
```



### Find **package by file**: `apt-file find` or `dpkg -S`

=== "apt-file"
    ```bash
    $ apt-file find psql | grep so
    libqt4-sql-psql: /usr/lib/x86_64-linux-gnu/qt4/plugins/sqldrivers/libqsqlpsql.so
    ```

=== "dpkg"
    ```bash
    $ dpkg -S /usr/bin/nvidia-container-cli
    libnvidia-container-tools: /usr/bin/nvidia-container-cli
    ```


### Find **files by package**: `apt-file list` or `dpkg -L`

=== "apt-file"
    ```bash
    $ apt-file list davfs2
    davfs2: /etc/davfs2/davfs2.conf
    ...
    ```

=== "dpkg"
    ```bash
    $ dpkg -L puppet-agent
    /etc/default
    ...
    ```


### **Add a repo**

```bash
sudo add-apt-repository "deb http://us.archive.ubuntu.com/ubuntu/ saucy universe multiverse"
```



### **List all active repositories**

```bash
grep -v '^$\|^\s*\#' /etc/apt/sources.list /etc/apt/sources.list.d/*
```



### **Remove a repo**

```bash
sudo apt-add-repository --remove "deb http://llvm.org/apt/utopic/ llvm-toolchain-utopic main"
```

Remove both --- packages installed from a repo and repo itself:
```bash
sudo ppa-purge ppa:webapps/preview
```



### **Securely add repo keys**
Source articles: [Ubuntu.SecureApt](https://help.ubuntu.com/community/SecureApt),
[Debian.SecureApt](https://wiki.debian.org/SecureApt).

Checking trust path:

```bash
$ GET https://download.owncloud.org/download/repositories/8.2/Ubuntu_15.10/Release.key | gpg --import
$ gpg --check-sigs --fingerprint 5180350A
pub   2048R/5180350A 2015-10-08 [expires: 2017-12-16]
      Key fingerprint = BCEC A903 25B0 72AB 1245  F739 AB7C 32C3 5180 350A
uid                  ce OBS Project <ce@s2.owncloud.com>
sig!3        5180350A 2015-10-08  ce OBS Project <ce@s2.owncloud.com>

1 signature not checked due to a missing key
```

Now you can verify and check imported key info, download other keys, for example:

```bash
$ gpg --list-sigs --fingerprint 5180350A
pub   2048R/5180350A 2015-10-08 [expires: 2017-12-16]
      Key fingerprint = BCEC A903 25B0 72AB 1245  F739 AB7C 32C3 5180 350A
uid                  ce OBS Project <ce@s2.owncloud.com>
sig 3        5180350A 2015-10-08  ce OBS Project <ce@s2.owncloud.com>
sig 3        479BC94B 2015-10-08  [User ID not found]


$ gpg --recv-keys 479BC94B
gpg: requesting key 479BC94B from hkp server keys.gnupg.net
gpg: key 479BC94B: public key "ownCloud build service <obsrun@localhost>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)

$ gpg --check-sigs --fingerprint 5180350A
pub   2048R/5180350A 2015-10-08 [expires: 2017-12-16]
      Key fingerprint = BCEC A903 25B0 72AB 1245  F739 AB7C 32C3 5180 350A
uid                  ce OBS Project <ce@s2.owncloud.com>
sig!3        5180350A 2015-10-08  ce OBS Project <ce@s2.owncloud.com>
sig!3        479BC94B 2015-10-08  ownCloud build service <obsrun@localhost>
```

and then I check the trust path from my key to at least one of the keys used to sign the archive key.
Only if I find an acceptable path will I then tell APT to trust the key:

```bash
$ gpg --export -a 5180350A | sudo apt-key add -
```
