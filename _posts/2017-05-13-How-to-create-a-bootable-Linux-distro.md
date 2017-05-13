---
published: true
author: Rubaiat
---
Linux is the system of choice for most real developers since it's minimal resource usage, lightning fast operations and tons of conveniences built to develop programs. It can be carried in a separate flash drive to boot the system anywhere someone want. In this post I'm gonna show how to create a bootable Linux Operating System.

For this process I'm gonna use an existing linux system and a 8 gb USB drive. You should get a minimum 4 gb drive. Users of Windows have to do it in other way, which I'll cover in the near future.

Now, if you have already tried instructions from the internet and tried to do this, chances are you've already encountered the classic **SYSLINUX** error. I've seen many great step by step instructions avoiding this and when you encounter this error, there's no consistent solution to this problem on the internet(_at least I've never seen one!_) The solution to this is that a **MBR injection**(_adding some additional instructions for the bootloader_) is needed.

This is a step by step guide, so watch carefully and do the following in order to get your own copy of a linux bootable distro!
 
1. First plug your USB and type in your terminal

{%highlight bash%}
# lsblk
{%endhighlight bash%}

This'll print all the devices mounted, the <USB_device> should be in the format **/dev/sdb** or **/dev/sdc**, not _/dev/sdb1_ or _/dev/sdc1_. **Caution:** Check the device name carefully or you might cause data erase from unwanted drive.

2. Download the ISO image of the distro of your choice. ISO images are basically the components of this distro bundled together as a single file.

3. Now for injecting the MBR we need a tool called **isohybrid**. You can get this by typing in your terminal

{%highlight bash%}
# sudo apt-get install isohybrid
{%endhighlight bash%}

After isohybrid is installed you need to make the MBR injection with it. Type in your terminal

{%highlight bash%}
# isohybrid /path/to/ISO/image
{%endhighlight bash%}

4. At this point the ISO can be written to the USB. To do this enter

{%highlight bash%}
# dd if=/path/to/ISO/image of=/dev/<USB_device> bs=2048 && sync
{%endhighlight bash%}

Note, that you need to specify the device name in place of the <USB_device>.
This step will take some time and won't print any output, so just wait and relax.

4. If no errors encountered and you see the prompt, then congratulations! You got your own copy of a linux bootable distro

The process is same for a dvd drive, USB's are more modular so it's great to have one or two of your favourite distros in USB to boot them anywhere you want. Kudos!
