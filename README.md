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

## Firewall and ssh
