title: ssh

# **ssh**

## **Reverse ssh tunnel**

Let's say you have a RaspberryPi without a static IP and a server with a static IP,
and you want to ssh from the server to your RaspberryPi.

Run the following command on RaspberryPi:
```bash
ssh -N -R 32003:localhost:22 tarpi@<ip-of-server>
```

It will bring `localhost:22` of RaspberryPi to `<ip-of-server>:32003`, so that you can ssh to
RaspberryPi with `ssh localhost -p 32003`


Consider creating a [restricted ssh user for port-forwarding](https://askubuntu.com/questions/48129/how-to-create-a-restricted-ssh-user-for-port-forwarding)
(only).



## **SSH port forwarding: bastions and `ProxyJump`'s**

Sometimes there are "bastion" hosts: in order to ssh to `host2`, you have first have to ssh to an intermediate `host1`.

There is proper support for this workflow in ssh: read more about `-J`, `-W` flags and `ProxyCommand`
[here](https://www.redhat.com/sysadmin/ssh-proxy-bastion-proxyjump).





## **Arbitrary port forwarding**
Suppose you have a web-server on `host2`, and this host is unreachable directly from your machine (`localhost`).
The only way to reach `host2` is through intermediate host - `host1`. And you want to be able to open pages from `host2`
from a browser running on your `localhost`.

To achieve this, just run it:
```bash
ssh -L  12345:host2:80 host1
```
(where `host2` is directly unreachable host, `host1` - intermediate host)

Once you run it, you can navigate to `http://localhost:12345` in your browser.



## **Agent forwarding**

You might want to maintain and use a single SSH key, regardless of how many nested SSH sessions there are. By default,
the key of the current localhost is used. Agent forwarding makes it possible to use a single ssh key, forwarding it
from one remote host to another. (1)
{.annotate}

1. Both ssh-agent as well as gpg-agent implement ssh-agent protocol. This secction describes only ssh-agent.

    See [Generating ssh keys -> GPG keys](ssh.md#generating-ssh-keys) for more info about gpg for ssh.


* Ensure local ssh config allows in:

    ```ini title="sudo nano /etc/ssh/ssh_config"
    Host *
    ForwardAgent yes
    ```

* Ensure remote server allows it:

   ```ini title="nano /etc/ssh/sshd_config"
   AllowForwardAgent yes
   ```

* Check configuration is fine: `/usr/sbin/sshd -t`
* And restart sshd: `service ssh restart`
* Start ssh-agent automatically:

    ```bash title=".bashrc"
    eval `ssh-agent`
    ```




## **Escape characters**

`ssh` has [these](https://man7.org/linux/man-pages/man1/ssh.1.html) escape characters:

* `~.` --- disconnect (useful if it hang).
* `~^Z` --- background ssh.
* `~#` --- list forwarded connections.
* `~?` --- display a list of escape characters.
* `~C` --- open command line.

    Currently this allows the addition of port forwardings using the -L, -R and -D options.
    It also allows the cancellation of existing portforwardings with `-KL[bind_address:]port` for local,
    `-KR[bind_address:]port` for remote and `-KD[bind_address:]port` for dynamic port-forwardings.
    `!command` allows the user to execute a local command if the `PermitLocalCommand` option is enabled in ssh_config.



## **Getting ssh fingerprint**

When you ssh to a host first time, you are supposed to verify its key fingerprint.

You can obtain host's fingerprint in this way:

```bash
ssh-keyscan localhost 2>/dev/null | ssh-keygen -lf -
```

## **Prevent dropping idle connections**

[SO](https://superuser.com/questions/699676/how-to-prevent-ssh-from-disconnecting-if-its-been-idle-for-a-while)

The problem is that there is something (usually a firewall or load-balancer), which is dropping idle sessions.
If you configure session keepalives, the keepalives will prevent network devices from considering the session as idle.

```title="sudo nano /etc/ssh/sshd_config"
ClientAliveInterval 10
ClientAliveCountMax 10
```



## **Hardenining** --- Publickey-only auth, disable passwords

```title="sudo nano /etc/ssh/sshd_config"
Protocol 2
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
MaxAuthTries 3
PermitRootLogin no
```

Or, for specific user:

```title="sudo nano /etc/ssh/sshd_config"
Match User <user_name_here>
   PasswordAuthentication no
```

and then: `sudo systemctl restart sshd`


[A very nice tutorial](https://blog.stribik.technology/2015/01/04/secure-secure-shell.html) on hardening SSH
Server that includes things like putting SSH in TOR.




## **SSH + 2FA**

[You can use various 2FA apps](https://www.digitalocean.com/community/tutorials/how-to-set-up-multi-factor-authentication-for-ssh-on-ubuntu-16-04)
(such as Google Authenticator) with SSH.




## **sshfs**

##### auto-reconnect

`sshfs` allows you to mount remote computer's filesystem over network. But sometimes the connection gets broken.

To mitigate it, you can mount in such way:

```bash
sshfs impedance:/mnt/media /mnt/media/ -o reconnect,ServerAliveInterval=1,ServerAliveCountMax=2
```

##### fstab

Example of `fstab` for `sshfs`:
```bash
sshfs#<login>@<server>:/remote/path/ /local/path fuse IdentityFile=/home/<login>/.ssh/id_ecdsa,idmap=user,allow_other 0 0
```



## **Generating ssh keys**

##### Elliptic curves

```bash
ssh-keygen -t ed25519 # Or, another curve: ssh-keygen -t ecdsa -b 521
```

Read more about [ed25519](https://security.stackexchange.com/questions/50878/ecdsa-vs-ecdh-vs-ed25519-vs-curve25519).

##### Keeping keys on a harware token (aka Yubikey)

??? "Advantages of Yubikeys (in comparison with storing ssh keys on a filesystem)"

    * Keys are never stored on filesystem of your PC.
    * It is [nearly impossible](https://security.stackexchange.com/questions/93399/how-can-it-be-easy-to-write-but-impossible-to-extract-the-private-key-from-a-c/93415#93415)
      to extract (private) keys from Yubikey (and impossible without your noticing it).
    * Yubikey has a hardware-backed max PIN entry counter (for example, your LUKS password can be brute-forced
      arbitrarily long, but Yubikey limits the number of attempts to 3).
    * Yubikeys are "detachable" and can be stored in a safe/remote place.
    * You do not need to insert it physically in a USB because there are NFC versions.

There are several ways to make ssh work with keys on a Yubikey:

=== "natively via FIDO/U2F-backed keys"

    ??? warning inline end "FIDO/U2F-backed ssh keys security concerns"

        Note, however, that FIDO/U2F has different security properties:

        * Often, _only_ touch is neede to use the key, unlike gpg, where you have to enter PIN
          code and and which limits number of failed attempts.
        * It is likely that touch will be needed for each ssh-connection (which might be a problem,
          if you are using ssh in a loop/script), unlike gpg, which caches your PIN for some time.

    [Recent ssh versions](https://www.openssh.com/txt/release-8.2) can generate FIDO/U2F token-backed
    keys. This options is good when you do not need/want to create "proxy" GPG keys and if
    remote host (where you are going to ssh to, for example github) supports such keys.


    In order to generate ssh keys directly on FIDO/U2F token, use `-t ed25519-sk` or `-t ecdsa-sk`:

    ```bash linenums="1"
    ssh-keygen -t ed25519-sk -O verify-required -f ~/.ssh/id_ed25519_sk
    ```

    * `-O verify-required` makes the token require PIN (safer but more annoying)
    * `-O resident` will put key handle to token. Without the option, the token alone is not usable.



=== "or through 'proxy' GPG keys"

    The idea is that you create GPG keys, put them on a Yubikey, and ask `gpg-agent` to "pretend" to be `ssh-agent`.
    This option is good when you already have gpg keys.

    1. Setup Yubikey as a SmartCard (_optional_):
        1. Buy an [appropriate](https://github.com/DimanNe/secure-boot#buy-equipment) NFC card-reader
           (for example `HID Identity OMNIKEY 5422 Internal USB 2.0 Grey Smart Card Reader`).
        2. Blacklist standard Linux drivers (source: [1](https://wiki.archlinux.org/index.php/Touchatag_RFID_Reader),
           [2](https://oneguyoneblog.com/2016/11/02/acr122u-nfc-usb-reader-linux-mint/)):
            ```bash linenums="1"
            sudo rmmod pn533_usb pn533 nfc
            echo "install nfc /bin/false" | sudo tee -a /etc/modprobe.d/blacklist.conf
            echo "install pn533 /bin/false" | sudo tee -a /etc/modprobe.d/blacklist.conf
            ```
        3. Install `libnfc`: `sudo apt install libnfc-bin pcsc-tools`
        4. Verify it works: (re-)plug the NFC reader, put your Yubikey on it, scan with: `pcsc_scan` and run `gpg --card-edit`.

    2. Create GPG keys locally and move them to a Yubikey.
        * You can use [DrDuh's guide](https://github.com/drduh/YubiKey-Guide).
        * Or you can use [a script that automates it](https://github.com/DimanNe/secure-boot#generate-gpg-keys).

    3. Setup ssh<--->gpg interaction (main [source](https://opensource.com/article/19/4/gpg-subkeys-ssh)):
        1. Enable ssh support in `gpg-agent` config:

            === "bash"
                ```bash linenums="1"
                cat << HEREDOC > ~/.gnupg/gpg-agent.conf
                enable-ssh-support
                # default-cache-ttl 60 # Change if you wish
                # max-cache-ttl 120    # Change if you wish
                HEREDOC
                ```
            === "fish"

                ```fish
                echo "enable-ssh-support
                # default-cache-ttl 60 # Change if you wish
                # max-cache-ttl 120    # Change if you wish" > ~/.gnupg/gpg-agent.conf
                ```

        2. Restart `gpg-agent`: `gpgconf --kill gpg-agent`
        3. Specify which key should be used for ssh: add keygrip of the "Auth GPG key" to `~/.gnupg/sshcontrol`:
            * Get its keygrip: `gpg -K --with-keygrip`
            * Put it in ~/.gnupg/sshcontrol: `echo 87A342B1561ADD416AD21... > ~/.gnupg/sshcontrol`


        4. Tell SSH to use `gpg-agent`'s socket:

            === "bash"
                ```bash title="~/.bashrc" linenums="1"
                # This makes ssh-agent use gpg-agent (to lookup available keys).
                # More info see here: https://opensource.com/article/19/4/gpg-subkeys-ssh
                export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
                gpgconf --launch gpg-agent
                ```

            === "fish"
                ```bash title="~/.config/fish/config.fish" linenums="1"
                # This makes ssh-agent use gpg-agent (to lookup available keys).
                # More info see here: https://opensource.com/article/19/4/gpg-subkeys-ssh
                set -x SSH_AUTH_SOCK (gpgconf --list-dirs agent-ssh-socket)
                gpgconf --launch gpg-agent
                ```

    4. Verify it works. `ssh-add -L` should show you public ssh-keys (backed by Yubikey & gpg).
