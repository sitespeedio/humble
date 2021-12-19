# Humble - Raspberry Pi WiFi network link conditioner

Connect your Raspberry Pi4 with your router turn on the Pi and join your new configurable WiFi network named `humble`. Access your Raspberry Pi4 through HTTP and choose the WiFi speed like this:

![throttle](https://user-images.githubusercontent.com/540757/146691181-4bef0437-fa58-41a4-a5ae-1d12da7bbf05.gif)

Table of Contents
=================
* [Background](#background)
* [Prerequisite](#prerequisite)
* [Install using the pre-made image](#install-using-the-pre-made-image)
  * [Log into the device](#log-into-the-device)
  * [Changing WiFi password](#changing-wifi-password)
  * [Changing SSH login password](#changing-ssh-login-password)
  * [Use a USB WiFi antenna](#use-a-usb-wifi-antenna)
  * [Raspberry Pi image setup](#raspberry-pi-image-setup)
  * [Change WiFi geographical location](#change-wifi-geographical-location)
  * [Turn on auto update for the Throttle front end](#turn-on-auto-update-for-the-throttle-front-end)
* [Install from scratch](#install-from-scratch)
  * [Install the OS and prepare your device](#install-the-os-and-prepare-your-device)
  * [Enable WiFi](#enable-wifi)
  * [Install and setup](#install-and-setup)
  * [Install throttle frontend](#install-throttle-frontend)

## Background
Inspired by Sam Smiths [PiNC](https://github.com/phuedx/pinc) I wanted to make a easy way for everyone (not just developers) to try out different internet speeds.

## Prerequisite
You need a Raspberry Pi 4 with a Ethernet cable that you can connect to your router.

## Install using the pre-made image
1. Download the pre-made image from [releases](https://github.com/sitespeedio/humble/releases).
2. Burn the image on a SD card using [Raspberry Pi Imager](https://www.Raspberry Pi.com/software/).
3. Insert the SD card into the Raspberry Pi 4.
4. Connect your Raspberry Pi with your router through a Ethernet cable and turn on your Raspberry Pi.
5. Join your new configurable `humble` WiFi network on desktop or mobile phone with the password `goslowtogoFAST`.
6. Access your Raspberry Pi through HTTP and chose the WiFi speed. On Mac access `http://Raspberry Pi.local:3001` from your web browser. On other OS you can use `nmap` to [find your Raspberry IP address](https://community.wia.io/d/11-how-to-set-up-a-raspberry-pi-without-an-external-monitor-or-keyboard). Access that IP using port 3001 in your browser.
7. Choose the WiFi speed.

When you checked that everything works you should change the WiFi password and the SSH password!

### Log into the device
Use the user `pi` and password `myloveisyourloveforever` to log into the device using SSH.

### Changing WiFi password

Edit the file `/etc/wpa_supplicant/wpa_supplicant.conf` and change the password to your new password.

```bash
sudo nano /etc/hostapd/hostapd.conf
```

Reboot by `sudo reboot`.

### Changing SSH login password
Login to the device and change the password to your new password using `passwd`.

### Use a USB WiFi antenna
If you want to use your *humble* installation for performance testing you probably want to use a USB WiFi antenna that is better than the default one that comes with the Raspberry Pi. 

The new WiFi will probably be `wan1` on your Raspberry and that means you need to reconfigure your Pi so its used:

1. Edit */etc/dhcpcd.conf* with `sudo nano /etc/dhcpcd.conf` and change `wan0` to `wan1`.
2. Edit */etc/dnsmasq.conf* with `sudo nano /etc/dnsmasq.conf` and change `wan0` to `wan1`.
3. Edit */etc/hostapd/hostapd.conf* with `sudo nano /etc/hostapd/hostapd.conf` and change `wan0` to `wan1`.
4. Reboot by `sudo reboot`.

### Raspberry Pi image setup
The image (based on Raspberry Pi OS Lite) has the following extra setup:

* It runs unattended-upgrades to keep the image up to date.
* It runs fail2ban
* It starts [the web version of throttle](https://github.com/sitespeedio/throttle-frontend) on startup using systemd.


### Change WiFi geographical location 
The WiFi setup in the images has Sweden as geographical setup and you probably want to change that. Use https://en.wikipedia.org/wiki/ISO_3166-1 to find your country code. I'm located in Sweden so I use `SE`.

You need to change in two different places. First run:

```bash
sudo raspi-config nonint do_wifi_country SE
```

And then edit the file `/etc/hostapd/hostapd.conf` and change the current country code to yours.

```bash
sudo nano /etc/hostapd/hostapd.conf
```

And change the row:
```
country_code=SE
```

Then reboot by `sudo reboot`.
### Turn on auto update for the Throttle front end

You can make sure user interface is automatically updated by adding the following in the crontab.

First open the crontab:
```bash
crontab -e
```

And then add the following lines:
```
0 6 * * *  bash -lc /home/pi/update.sh  2>&1 | logger -t humble-updates
```

## Install from scratch

Thank you SpaceRex for your tutorial [Turn your Raspberry Pi into a WiFi Router!](https://www.youtube.com/watch?v=laeOmNDE-Ac), that helped me a lot to set this up! Checkout the [video](https://www.youtube.com/watch?v=laeOmNDE-Ac) for more details.

### Install the OS and prepare your device
1. Download and install the latest [Raspberry Pi OS Lite](https://www.Raspberry Pi.com/software/operating-systems/) image to a SD card using [Raspberry Pi Imager](https://www.Raspberry Pi.com/software/).
2. Add an empty file named `SSH` to root section of the SD card so you more easily can access the Raspberry Pi.
3. Insert the SD card into the Raspberry Pi, connect your Raspberry Pi with your router through a Ethernet cable and turn on your Raspberry Pi.
4. Find the IP number of your Raspberry Pi, use `nmap` to [find your Raspberry IP address](https://community.wia.io/d/11-how-to-set-up-a-raspberry-pi-without-an-external-monitor-or-keyboard) or on Mac just use `Raspberry Pi.local`.
5. Login to your Raspberry using SSH (this example if for Mac): `SSH pi@Raspberry Pi.local` with password `raspberry`.
6. Once you logged in, change the password for the *pi* user using `passwd` to set a new password.

Now you are ready to turn your Raspberry Pi into a WiFi router.

### Enable WiFi
1. By default the WiFI on the Raspberry Pi is disabled (you will get a message like **Wi-Fi is currently blocked by rfkill.** when you login to the device). Enable the WiFi by setting your geographical location. Use https://en.wikipedia.org/wiki/ISO_3166-1 to find the country code. I'm located in Sweden so I use `SE`.

First run:
```bash
sudo raspi-config nonint do_wifi_country SE
```
Then reboot the device:
```bash
sudo reboot
```

### Install and setup
1. First make sure to update:
```sudo apt update```
2. Install the following so your Raspberry Pi can act as a router: 
```sudo DEBIAN_FRONTEND=noninteractive apt install -y hostapd dnsmasq netfilter-persistent iptables-persistent```
3. Make sure it starts on boot:
```bash
sudo systemctl unmask hostapd.service
sudo systemctl enable hostapd.service
```

4. Edit `etc/dhcpcd.conf` and make sure to add the interface at the bottom of the file and set the subnet that is gonna be used (make sure you don't use that IP before on your network). This makes sure that when wlan0 is used (our WiFi) the Raspberry Pi get IP 10.2.1.1:
```bash
sudo nano /etc/dhcpcd.conf
```

Then add at the bottom of the file:
````
interface wlan0
    static ip_address=10.20.1.1/24
    nohook wpa_supplicant

````

5. Then edit `/etc/sysctl.d/routed-ap.conf` to forward ipv4 adresses.

Open the file: 
```bash
sudo nano /etc/sysctl.d/routed-ap.conf
```
and add `net.ipv4.ip_forward=1` in that file.

6. Tell the traffic where to go. In the terminal run:

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo netfilter-persistent save
```

6. The step prepare your Raspberry tp be a router and setup which IP addresses devices will have when they connect. That happens on `/etc/dnsmasq.conf`. 

As a first step, move the old version so we have a backup:

```bash
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.old
```

Then open (a new) file:
`sudo nano /etc/dnsmasq.conf` 

And add the following lines:
```
interface=wlan0
dhcp-range=10.20.1.5,10.20.1.100,255.255.255.0,72h
domain=wlan
address=/rt.wlan/10.20.1.1
```

7. Setup your host access point file. Edit `/etc/hostapd/hostapd.conf`
Here you need to add your country code again (https://en.wikipedia.org/wiki/ISO_3166-1) and set a password (`wpa_passphrase`) and a name `ssid` of your WiFi network.

Open the file:
```
sudo nano /etc/hostapd/hostapd.conf
```

And then add:
```                     
country_code=SE
interface=wlan0
ssid=humble
hw_mode=a
channel=36
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=goslowtogoFAST    
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

Change the `country_code` to your country code. 

It's also in this configuration file you choose what frequency your WiFi network will be on.     
`hw_mode`can be either *a* = 5GHz, *b* = 2.4Ghz or *g* =2.4 GHz.

8. The WiFI setup is done. Reboot your Pi `sudo reboot` and try to connect to your WiFi network on your desktop or mobile.

### Install throttle frontend
To be able to run the frontend you need to install NodeJS. [Install latest LTS](https://nodejs.org/en/), when I write this that version is 16.13.1. 

```bash
wget https://nodejs.org/dist/v16.13.1/node-v16.13.1-linux-armv7l.tar.xz
tar xf node-v16.13.1-linux-armv7l.tar.xz
cd node-v16.13.1-linux-armv7l/
sudo cp -R * /usr/local/
```

Then you can verify that it worked by running:
```bash
node --version
```

As the last step you need to install the frontend.

```bash
sudo apt install -y git
git clone https://github.com/sitespeedio/throttle-frontend.git
cd throttle-frontend
npm install --production
```

Start the frontend with: 
```bash
node lib/app.js
```

And open your browser and access ```http://<Raspberry Piip>:3001```
