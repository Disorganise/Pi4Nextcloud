# Pi4Nextcloud
Running [Nextcloud](https://nextcloud.com/) &amp; more on RPi4

Inspiration from;  
https://github.com/chrisbeardy/nextcloud-docker-raspberrypi-tutorial  
https://eligiblestore.com/blog/2020/10/05/complete-guide-rasp-pi-4-for-nextcloud-redis-mysql-ext-ntfs-nginx-proxy-manager/

# Installing Nextcloud on Raspberry Pi4 with docker
[This is initially a collection of notes - with the intent to clean up later]

## Background
I've fumbled around creating and re-creating Pi configs, only to forget exactly what I did to get there.  So this is intended primarily as a note to myself for future reference.  I'm making it public in case it's useful for anyone else.
I've been running Nextcloud (and prior to that, owncloud) for many years as Hyper-V VMs.  I used images from [Hansson IT](https://www.hanssonit.se/nextcloud-vm/) both free and purchased, and they served me well.  The last rebuild I did was around 2020 iirc and I mounted the ncdata via SMB - it seemed a good idea at the time but it also caused me some issues with files 'disappearing' from time to time.  Now I'm 'stuck' with updating to the latest Nextcloud due to the older PHP version and so (finally) I'm getting around to using my Pi4 for its intended purpose.

## Boot from USB
I've a couple of microSD cards become corrupt on me, likely due to the power cuts we see.  The SD cards are too small to be really useful for Nextcloud anyway, so any external USB hard drive will suit the job better.

According to [this article](https://www.pragmaticlinux.com/2021/12/directly-boot-your-raspberry-pi-4-from-a-usb-drive/), most Raspberry Pi4's can boot from USB.  Mine's an 8GB version so I figured it was 'later'.  I originally planned to use the Ubuntu image, but had issues booting from the USB hard drive, so reverted to the Raspbian image.  
I did try updating the bootloader as mine was April 2020, but it still wouldn't boot the Unbuntu image even after updating to bootloader Jan 2023. 

I used the Raspberry Pi Imager to download and install 'Rasberry Pi OS (other)/Rasberry Pi OS Lite (64 bit)' onto the USB Hardrive - in my case it's a WD My Passport 2TB.   
Once the imager has installed the OS, disconnect the drive from the PC (remember to used eject) and plug the drive into one of the blue USB ports on the Pi.  [Note, if there are issues booting the USB, you can try the black USB ports, or first update the bootloader by booting Raspbian of a micros-SD card and then try booring from the USB after the bootloader is updated]

The steps for updating the bootloader follow:

### Updating bootloader
1. Install Rasperry Pi OS to micro-SD card/USB hard drive
2. Boot Raspberry OS
3. Run the commands;  
   `sudo apt update`  
   `sudo apt full-upgrade`  
4. Check if there's a bootloader update
   `sudo rpi-eeprom-update`  
5. Update the bootloader (if applicable)
   ` sudo rpi-eeprom-update -a`  
   ` sudo reboot`  
   The Pi should now reboot, detect that an eeprom update is pending, update it and then reboot again

# Installs

## Prep - Enable SSH
If you haven't already, it's probably a  good idea to enable the SSH server.  That will then allow you to connect to the Pi over the network rather than needing a keyboard and display to be plugged in.  
Run `sudo raspi-config'  
Go to Interface options/SSH and select yes to enable
![Imgur](https://i.imgur.com/z7cN4Vh.png)
![Imgur](https://i.imgur.com/vZHaH61.png)
![Imgur](https://i.imgur.com/y4qkmKV.png)
![Imgur](https://i.imgur.com/pEUThVR.png)

## Install DockSTARTer
`sudo apt-get update`  
`sudo apt-get dist-upgrade`  
`sudo apt-get install curl git`  
`bash -c "$(curl -fsSL https://get.docker.com)"`  
`bash -c "$(curl -fsSL https://get.dockstarter.com)"`  
`sudo reboot`  

# Configure DockSTARTer
`ds`  
I initially added Portainer and Watchtower.  Watchtower is selected by default. I said no to the config as the timezone was incorrect, aside from changing that and hostname to match the system name I'd chosend, I accepted the defaults and let it run the compose.  The containers downloaded and started running (check by running `docker ps`)

# Reverse Proxy
