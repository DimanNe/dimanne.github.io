title: usb & udev


# **USB & UDEV**

## **USB**

* **List usb devices**: `#!fish lshw`, `#!fish lsusb # -vvv`, `#!fish usb-devices`. GUI: `sudo usbview`.
* **Get paths of a device**: When you insert a device a number of paths on filesystem gets associated with it.
  Here is how you can find all of them.

    * While running: `udevadm monitor --property`
    * Insert your device, and find string like `DEVNAME=/dev/bus/usb/003/006` in the output.

* **Get extended permissions**: `getfacl /dev/bus/usb/001/002`




## **udev**

Docs: [debian.org/udev](https://wiki.debian.org/udev),
[freedesktop.org multiseat](https://www.freedesktop.org/wiki/Software/systemd/multiseat/),
[intro](https://opensource.com/article/18/11/udev).

### **Config file discovery rules**

* `/run/udev/rules.d`: supposed to for overwriting by administrator
* `/etc/udev/rules.d`: supposed to for overwriting by administrator
* `/lib/udev/rules.d`: Packages install rules here.

If a file with the same name is present in more than one of these directories then 1st one wins.

Files in there are parsed in alphabetical order, as long as the name ends with `.rules`.

### **Attributes by path**

Once you have a device path (see above), you can:

* **Get canonical path**: `#!fish udevadm info -q path -n /dev/bus/usb/001/002` will print
  `/devices/pci0000:00/0000:00:01.2/0000:02:00.0/0000:03:08.0/0000:05:00.1/usb1/1-1`.

* **Get all its attributes**:
    * `#!fish udevadm info -a -n /dev/bus/usb/001/002` or
    * `#!fish udevadm info -a -p (udevadm info -q path -n /dev/bus/usb/001/002)`

* **Simulate rule execution**: `udevadm test (udevadm info -q path -n /dev/bus/usb/003/006)`


### **Example**

Example of running a script on new flash-drive insertion.

Add the following file:

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




### **Example of removing tags uaccess and seat tags**
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


### **Example 3**

```title="sudo nano /usr/lib/udev/rules.d/91-keyboard.rules"
SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0113|0114|0115|0116|0120|0200|0402|0403|0406|0407|0410", ACTION=="add", GROUP="seckey" SYMLINK+="myusbkey"
```
