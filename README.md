# SD Card Setup

## 0. Create Raspi image

Can choose from many OS's (Ubuntu Server 64-bit)

Use [Raspberry Pi Imager](https://www.raspberrypi.org/software/)

## 1. Enable remote login

### Raspbian

Create (`touch`) a file in the `boot` folder (`/media/edward/boot`) named `ssh`

### Ubuntu Server

This is already enabled by default on Ubuntu Server.

## 2. Enable Wifi

### Raspbian

Put a `wpa_supplicant.conf` in the `boot` folder that looks like this:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
	ssid="$WIFI NAME"
	psk="$WIFI PASSWORD"
	key_mgmt=WPA-PSK
}
```

### Ubuntu Server

[Instructions Here](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#3-wifi-or-ethernet)

```
# NOTE: Be sure to enclose network name in quotation marks!!
# "Note: During the first boot, your Raspberry Pi will try to connect to this network. It will fail the first time around. Simply reboot sudo reboot and it will work.
# NOTE: This only works on first boot! After that, see `/etc/netplan/50-cloud-init.yaml` (https://linuxconfig.org/ubuntu-20-04-connect-to-wifi-from-command-line, https://huobur.medium.com/how-to-setup-wifi-on-raspberry-pi-4-with-ubuntu-20-04-lts-64-bit-arm-server-ceb02303e49b)

nano /media/edward/system-boot/network-config
```

*Note: `sudo apt-get` does not work on first boot for some reason.*

# Setup Over WiFi: SSH

After booting the Raspi using the SD Card, login w/ pi@<ip> (password: `raspberry`)

0. Set hostname

```
/etc/hostname
```

1. Setup SSH Key

```
mkdir ~/.ssh
cd ~/.ssh
ssh-keygen -t rsa -b 4096
```

Add key to `~/.ssh/authorized_keys` file!!!

2. Disable Remote Password Login

First test that remote login works without a password prompt!

```shell
sudo nano /etc/ssh/sshd_config
```
Update the following line:
```shell
PasswordAuthentication no
```
Then exit and
```shell
sudo service sshd restart
```

3. Add SSH Key to Git and checkout `raspi`

4. Set up ssh reverse tunnel to work without passwords for a remote droplet

5. Set up autossh to establish reverse tunnel on boot

Install autossh
```shell
sudo apt-get install autossh
```

Add the following line to `/etc/rc.local`
```shell
sudo nano /etc/rc.local
```
```shell
su pi -c "bash /home/pi/autossh.sh" &
```
The script `/home/pi/autossh.sh` should read:
```
{
  date
  whoami
  sleep 20
  echo connect
  autossh -N -f -i /home/pi/.ssh/id_rsa -R 22222:localhost:22 -R 8888:localhost:80 root@mooseisplural.com

  pushd .
  cd /home/pi/raspi/
  python3 lamp.py &>lamp.out &
  popd
} &>> /home/pi/autossh-out.log
```

6. Set timezone

```shell
sudo timedatectl set-timezone America/Chicago
```

# Google API Setup

### Calendar

https://developers.google.com/calendar/quickstart/python

### Re-authorize

1. Delete `token.pickle` file
2. Run script. Script should prompt w/ a link. Copy link and open in browser.
3. Follow auth flow ("proceed unsafely"). Auth flow will end by redirecting to a `localhost:<port>` link.
4. DO NOT KILL GCal script (this script is hosting the `localhost:<port>` server). Login to Raspi a second time. `curl` the `localhost:<port>` link - be sure to put the URL in quotations, otherwise, `curl` will incorrectly split up the link.
5. GCal script should complete normally.

# Nginx web server

## PHP
```
sudo apt-get install php7.1-fpm
sudo nano /etc/nginx/sites-available/default
```

[Configure PHP](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-in-ubuntu-16-04#step-4-configure-nginx-to-use-the-php-processor)

In `/etc/nginx/sites-enabled/default`, change php `.sock` path to `/run/php/php7.1-fpm.sock`.

Debug with
```
tail /var/log/nginx/error.log
```

# Spotify

```
https://pimylifeup.com/raspberry-pi-spotify/#:~:text=Using%20the%20Raspotify%20software%20package,connecting%20any%20speakers%20to%20Spotify.
```

## Audio Jack Select

```
sudo raspi-config
```

## Volume

```
alsamixer
```

# Conda

## Create new env

```
conda create -n raspi python=3.7.4
```

# Kiosk Mode

*Put `DISPLAY=:0` in front of your commands!

[hide cursor](https://www.raspberrypi.org/forums/viewtopic.php?t=234879)
[steal from the script to disable sleeping and update chromium settings](https://die-antwort.eu/techblog/2017-12-setup-raspberry-pi-for-kiosk-mode/)

### Resolution

#### https://www.raspberrypi.org/documentation/configuration/hdmi-config.md

#### https://www.raspberrypi.org/documentation/configuration/config-txt/video.md

This is key:

`/boot/config.txt`
```
hdmi_ignore_edid=0xa5000080
```

Then use the chart in the second link to pick a resolution.

### Overscan

(Didn't actually need to do this)

sudo raspi-config
>> Advanced Options
>> Overscan -> No
