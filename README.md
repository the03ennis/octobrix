# octobrix
Dockerized octoprint multi-instance environment with multi-webcam support.

## Context :

In the scope of using octoprint with my 3d printers (I own 4 printers), I was reluctant to buy 4 raspi + cam and set them (for time and cost).
I went for a solution using an old computer, usb cameras and a powered usb port to plug the printers and the cameras.
Main idea was to set up a docker server on the computer, create an octoprint container for each printer I own and plug printers + cameras on the usb hub. 

## Faced issues :

- Need static /dev links for printers devices : use a custom udev conf file.
- Need static /dev links for cameras : use a custom udev conf file.
- Cannot make more than one low cost usb camera works on the same usb port : need to patch uvc to prevent kernel allowing all bandwidth for a not well designed (cheap) camera. 

## Tested configuration :

- Computer : old gigabyte BRIX J1900 (GB-BXBT-19), 8Gb RAM, SSD 240Gb , Ubuntu server 20.04.2 LTS
- Usb hub : Powered, 10 ports, Acasis
- Usb cameras : MJPG (compressed) stream (not YUV only ! they won't work) 

## Installation procedure :

- Get the hardware 
- Install Ubuntu server 20.04.2 LTS on your computer
- Get octobrix from git repository and run install.sh (warning it involves a kernel recompilation, it tooks 12 hours for me)
- Finalize installation (edit udev configuration and stack.yml to map your printers)
- Start the containers

### Serial/USB Configuration
In order to ensure each printer goes to the correct octoprint instance each time, you need to configure custom interfaces for the USB ports via UDEV rules. 
The 3D Printers tend to connect as /dev/ttyACM* but they may also be /dev/ttyUSB*. 

The format is in 99-usb-serial.rules, but it looks like this:
`SUBSYSTEM=="tty", ATTRS{serial}=="CZPX1919X004XC24974", SYMLINK+="printer1"`

The serial here can be found with the following steps:

- The command `ls -al /dev/tty*` will show a list of all of the tty interfaces. Find the ones with ACM or USB.
- `udevadm info -a -n /dev/tty[xxx]` will dislay output related to the device, look through fields like vendor and product to ensure that you have the right device.
- grab the value listed for ATTRS{serial} and place it in the 99-usb-serial rules, changing the symlink so you don't have overlapping numbers at the end.
- copy the file to where it belongs: `/etc/udev/rules.d/99-usb-serial.rules`

### IPV6 Configuration

IPv6 configuration is *a LOT* simpler than I initially thought. All one needs to do is add the static ipv6 address to the netplan configuration file and disable dhcp4. This is what your file should look like.

`/etc/netplan/00-installer-config.yaml`:
```yaml
network:
  ethernets:
    enp4s0f0:
      dhcp4: true
      dhcp6: false
      addresses:
        - 2607:f460:1833:1::18:0/64
  version: 2
```

Note that **nothing** needs to be done in docker to enable ipv6.


The sysctl file also needs to be updated - add these two lines to the bottom of your `/etc/sysctl.conf`:
```conf
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.all.disable_ipv6 = 0
```

## TODO list :

- I'm not expert in kernel patching and while i wanted just to patch and reinstall uvc, my script end up compiling the whole kernel ! (need an expert looking on that point)
- The uvc_video.c patch does not works on YUV only camera (the very cheap $3 camera found on aliexpress) even removing UVC_FMT_FLAG_COMPRESSED check, i'm pretty sure we can make it works with a better understanding of the code (suspecting others fields than dwMaxPayloadTransferSize need to be altered).
- Implement auto switch on/off through PSU Control
- Write a tutorial for custom udev rules (people knowing a little can check the /files/ directory and easily writes their own rules to map a static /dev item mapping usb hub port number) 

## Contact :

- If you want to contact me , or give a few bucks through paypal if you find this project usefull, my email is : galand.olivier.david@gmail.com 
- Also modified by Daniel Ennis (dennis@suffieldacademy.org)
