# Archlinux WLAN setup

I've purchased UGREEN WiFi 6 AX900Mbps but ther is a problem, archlinux doesn't have the drivers for this USB dongle.

## Step to fix:

Download drivers and dynamically load them in the kernel.

Conditions:
* Yay installed to manage AUR packages
* Using linux zen kernel

Steps:
1. Install the linux-zen-headers (if you are using any other kernel, download the proper headers):

    `sudo pacman -S linux-zen-headers`

2. Download and install the drivers for this device, the drivers can be found in AUR packages, install them with:

    `yay -S aic8800d80-dkms`

By this time, you should have a network interface tor you WLAN network (if you do `ip a` you should see somthing like: `wlan0`) you might need to connect 
and configure this interface to have a working connection.

## Post installation steps:

you will need to connect to your SSID (WiFi network), you can choose many alternatives (including graphic ones)
but I will stick to terminal programs, in this case use iwctl (iwd) to configure the connection.

* Install iw and iwd: `sudo pacman -S iw iwd`
* Search your SSID and connect WiFi:
    1. run `iwctl`
    2. inside new prompt type: `station list` to find you network interfaces
    3. list networks available for your interface `station <id> get-networks` (ie: `station wlan0 get-networks`)
    4. connect to your network: `station wlan0 connect <your_SSID>` and introduce yout password.

At this point you should have you device connected to your WiFi network.

## Next: run improvements.

If you want the device to connect to your WiFi even before the first user has log in the system, you might need to tweak the iwd setup.

First, be sure that iwd is up and running and get started with your system with: `sudo systemctl enable --now iwd`

Now, configure iwd to automatically connect to you WiFi network (SSID), in order to do that:

1. Modify this file: `sudo nvim /var/lib/iwd/<ssid>.psk` 
2. Enable to connect automatically and do not disable on power saving:
    2.1. Run: `sudo nvim /etc/iwd/main.conf`
    2.2. Put this on the file:
    
    ```
    [General]
    EnableNetworkConfiguration=true
    DisablePowerSave=true

    [Network]
    RoutePriorityOffset=300

    [Settings]
    AutoConnect=true
    ```

3. **Optional:** edit this file: `sudo nvim /var/lib/iwd/<your_SSID>.psk` it should already have the security config from previous connection, just add:
``
[IPv4]
Address=192.168.0.XX
PerfixLength=24
Gateway=192.168.0.1
``
Assuming your router (internet gateway is in IP 192.168.0.1) you can assign it a static IP adress to make it easier to find later on.

## Other improvements

At this point we are providing a static IP for this device with iwd (the low level daemon) but we could make use of systemd and systemd-networkd in particular
to take care of the IP assignment and DCHP configuration but for the moemnt I'll leave it as it is.
