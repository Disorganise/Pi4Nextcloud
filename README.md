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
Once the imager has installed the OS, disconnect the drive from the PC (remember to used eject) and plug the drive into one of the blue USB ports on the Pi.  [Note, if there are issues booting the USB, you can try the black USB ports, or first update the bootloader by booting Raspbian of a micros-SD card and then try booting from the USB after the bootloader is updated]

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

## Portainer
Portainer isn't necessarily required, but it provides a nice GUI for checking the status of docker containers.  
The install instructions can be found on the [Portainer Documentation](https://docs.portainer.io/start/install-ce/server/docker/linux) but it boils down to the following commands:  
Creation of a volume for the Portainer Server to store its database
```docker volume create portainer_data```  
And the actual installation command.  This is defaulting to a self-signed certificate for https  
```docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest```  

Connect to Portainer using https://<ip address/or hostname>:9443
You'll initially need to configure the username and password - note the username defaults to 'admin' but it is a good idea to change that.  
If you don't connect to the Portainer webpage soon enough, the initial config will time out and require you to restart the container.  
Do that by first grabbing the cointainer ID by using ```docker ps```  
The leftmost entry is the container ID.  
Copy that and issue the ```docker restart <container ID>``` command, obviously replacing <container ID> with the string you copied just before.


# Reverse Proxy
Since we have portainer installed, we'll use that to run the docker compose to install [Nginx Proxy Manager](https://nginxproxymanager.com/)
In the Portainer GUI, navigate to Stacks  
Tap +Add Stack  
Give it a meaningful name - e.g., npm-reverse-proxy  
Copy and paste the data from the /reverse-proxy/docker-compose.yml file.  
It's there as a separate file so it can be run directly using docker compose if your prefer.  
The file will create the Nginx Proxy Manager using a sqlite database  
Click 'Deploy the stack'

Connect to NPM using <ip/hostname>:81
The default credentials are  
Email:    admin@example.com  
Password: changeme  

You'll be required to immediately change these.

# Nextcloud
We're going to go with Nextcloud All-in-one.
Ultimate sources are:  The example [compose.yaml] (https://github.com/nextcloud/all-in-one/blob/main/compose.yaml)
and the info about [reverse procy] (https://github.com/nextcloud/all-in-one/blob/main/reverse-proxy.md) 

## Initial Deploy of AIO container
Go back to Portainer to create a new Stack
Go to Stacks, the +Add stack
Give it a meaningful name such as Nextcloud

Copy and paste the data from the /nc-aio/docker-compose.yml file.

Click 'Deploy the stack'

## Set up reverse proxy
Ensure you have a public domain name available and pointing to your public IP address.  I use https://freedns.afraid.org/, but there are many options.

Now head over to NPM  (i.e. <ip/hostname>:81)
Add a new Proxy Host
Enter the fully qualified domain name into the 'Domain Names' section (e.g., nextcloud.mydomain.com)
Leave the Scheme as http - if you change it to http you get errors
Add the IP address of the Raspberry Pi
The Forward Port needs to be match what was in the docker-compose.yml  (where it says - APACHE_PORT=11001).  The default _was_ 11000 but I initially had an error so changed it to 11001.  
Enable the Clock Common Exploits and Websockets support
Leave the access list as Publicly Accessible

The Customer locations tab can be left blank

On the SSL tab choose 'request a new SSL certificate
Enable the Force SSL, HTTP/2 Support and HSTS Enabled options

Finally, on the Advanced tab add the following to the Custom Nginx Configuration
``` client_body_buffer_size 512k;
proxy_read_timeout 86400s;
client_max_body_size 0; ```

Click Save and the NPM should go off and get a certificate from LetsEncrypt

Don't bother trying to click the domain yet, it's not ready - you'll either get an error or if lucky, a token back.

## Continue Nextcloud deployment
Open a new tab in your browser and go to <ip/hostname>:8080
You should hopefully get an initial welcome type page from Nextcloud AIO.  Grab the passphrase and use it to log in.
You should get an initial config page.
You'll need to enter the FQDN and some other information.  I opted out of all the additional modules - the AV isn't compatible with ARM and the rest don't really interest me.  You choose whatever you want.
After stepping through the set up you'll hit the download and start containers button.  This will take a while to complete.
I noticed that when I looked into Portainer, a whole bunch of new containers were running with a couple flagged as unhealthy for quite some time.  It's a bit annoying (to me) that these are all 'loose' rather than being under the nextcloud stack :(

When everything goes healthy, you should be just about ready.

Go back to the <ip/hostname>:8080 tab and click that reload button if necessary.
You should get a username and password on screen together with an 'Open you Nextcloud' button.  Clicking that should open a new tab using the FQDN - use the admin username and password to log in and follow your nose.