title: usb & udev


# **USB & UDEV**

## **USB**

* **List usb devices**: `#!fish lshw`, `#!fish lsusb # -vvv`, `#!fish usb-devices`. GUI: `sudo usbview`.




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


### udev actions & permissions

* Get extended permissions: `getfacl /dev/bus/usb/001/002`
* **Monitor for devices**: `udevadm monitor`
* **Get canonical path**: `#!fish udevadm info -q path -n /dev/bus/usb/001/002` will print `/devices/pci0000:00/0000:00:01.2/0000:02:00.0/0000:03:08.0/0000:05:00.1/usb1/1-1`.
* **Get all attributes**:
    * `#!fish udevadm info -a -p (udevadm info -q path -n /dev/bus/usb/001/002)` or
    * `#!fish udevadm info -a -n /dev/bus/usb/001/002`
* **Write a rule**

    ```title="sudo nano /usr/lib/udev/rules.d/91-keyboard.rules"
    SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0113|0114|0115|0116|0120|0200|0402|0403|0406|0407|0410", ACTION=="add", GROUP="seckey" SYMLINK+="myusbkey"
    ```

* `udevadm test (udevadm info -q path -n /dev/bus/usb/003/014)`


## Other

* Paths: `ls -al /devices/pci0000:00/0000:00:01.2/0000:02:00.0/0000:03:08.0/0000:05:00.3/usb3/3-3/3-3:1.0`

