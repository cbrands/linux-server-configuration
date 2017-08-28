# linux-server-configuration

## upgrading packages 
All packages were upgraded on 28th of august 2017 at 13:32 (Timezone westerneurope) with the following commands
```
sudo apt-get update
sudo apt-get upgrade
```
I got one remark that 
"A new version of /boot/grub/menu.lst is avalable, but the version installed currently has been locally modified. 

![warning screen](/screenshots/menulst.png "warning screen")

As I didn't know what amazon had locally configured I kept the original, which was default.

## ssh
* Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
* Reload SSH using `sudo service ssh restart`.
Now the big orange button connect to ssh no longer works as it assumes port 22.
so download the private key (see instructor notes) and put it inside $HOME/.ssh. Now set permissions with

```
chmod 600 $HOME/.ssh/<your keypair file>
chmod 700 $HOME/.ssh
```
Now login with
```
ssh -p 2200 -i <your home directory>/.ssh/<your keypair file> ubuntu@35.156.112.239
```

## Firewall
The firewall was configured by typing the following commands

```
# default block all incoming ports
sudo ufw default deny incoming
# default open all outgoing ports
sudo ufw default allow outgoing
# we had ssh configured on port 2200
sudo ufw allow 2200/tcp
# open http port
sudo ufw allow 80/tcp
# open ntp port
sudo ufw allow 123/udp
# enable firewall
sudo ufw enable
```
chaeck the firewall status by typing sudo ufw status

```
sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)    
```

## Configure Local timezone to UTC
Timezone can be set with `sudo dpkg-reconfigure tzdata`
* select 'None of the above'
* select 'UTC'
* select 'ok'

## Create user grader
```
sudo adduser grader
sudo nano /etc/sudoers.d/grader
```
This will open an empty file
type `grader ALL=(ALL:ALL) ALL`
save and exit

