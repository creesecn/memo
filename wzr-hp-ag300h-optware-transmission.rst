==========================================================================
    install optware & transmission on wzr-hp-ag300h's official dd-wrt
==========================================================================

reference
================
* `[Howto] Install Optware on Atheros units (such as WNDR3700) 
  <http://www.dd-wrt.com/phpBB2/viewtopic.php?t=86912&postdays=0&postorder=asc&start=0&sid=b9213c37e25ed7c5d775f277aba247aa>`_
* `TL-WR1043ND - Install Transmission <http://klseet.com/index.php/tp-link-ddwrt/tl-wr1043nd-ver18-atheros/install-transmission>`_

wzr-hp-ag300h router config
====================================
* update to official dd-wrt: wzrhpag300h-pro-v24sp2-19154
* prepare your wlan connection and wireless connection

install optware
====================================
1. Prepare the USB disk 
--------------------------
Create an ext3 partition using GParted for instance 

2. Configure DD-WRT 
---------------------------
Under Services->Services->Secure Shell: 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
* Enable SSHd 
* Click Apply Settings 

Under Services->USB: 
~~~~~~~~~~~~~~~~~~~~~~
* Enable Core USB Support 
* Disable USB Printer Support (enable it if you need printing support) 
* Enable USB Storage Support 
* Enable Automatic Drive Mount 
* Set Disk Mount Point to /mnt 
* Click Apply Settings 

3. Plug the USB drive into the router and reboot it 
-------------------------------------------------------------
* SSH into your box using user root and make sure, using mount, that your USB stick was mounted correctly (you can also check this on the web interface under Services->USB). 

4. Create and prepare necessary structure 
-----------------------------------------------
* SSH into your box using user root if not already done at previous step::

        cd /mnt 
        mkdir /sda_part1 
        cd /mnt/sda_part1 
        mkdir etc opt root 
        touch optware.enable 
        chmod 755 etc opt root 
        mkdir opt/lib 
        chmod 755 opt/lib 
        cp -a /etc/* /mnt/sda_part1/etc/ 
        mount -o bind /mnt/sda_part1/etc /etc 
        mount -o bind /mnt/sda_part1/opt /jffs 

5. Install the required libraries for the MIPS (big-endian) architecture and OpenWRT's opkg 
--------------------------------------------------------------------------------------------------
install opkg::
        cd /tmp 
        wget http://downloads.openwrt.org/snapshots/trunk/ar71xx/packages/libc_0.9.33.2-1_ar71xx.ipk 
        wget http://downloads.openwrt.org/snapshots/trunk/ar71xx/packages/opkg_618-6_ar71xx.ipk
        ipkg install libc_0.9.33.2-1_ar71xx.ipk opkg_618-6_ar71xx.ipk 

You will get the following output with error messages. You can't avoid it so don't worry about it.::

        ERROR: File not found: //usr/local/lib/ipkg/lists/whiterussian 
        You probably want to run `ipkg update' 
        ERROR: File not found: //usr/local/lib/ipkg/lists/non-free 
        You probably want to run `ipkg update' 
        ERROR: File not found: //usr/local/lib/ipkg/lists/backports 
        You probably want to run `ipkg update' 
        /bin/ipkg: line 1184: sort: not found 
        Unpacking libc...Done. 
        Configuring libc...Done. 
        ERROR: File not found: //usr/local/lib/ipkg/lists/whiterussian 
        You probably want to run `ipkg update' 
        ERROR: File not found: //usr/local/lib/ipkg/lists/non-free 
        You probably want to run `ipkg update' 
        ERROR: File not found: //usr/local/lib/ipkg/lists/backports 
        You probably want to run `ipkg update' 
        /bin/ipkg: line 1184: sort: not found 
        Unpacking opkg...Done. 
        Configuring opkg...Done. 

Type the following lines to create the configuration file for opkg:: 

        cat > /etc/opkg.conf << EOF 
        src/gz snapshots http://downloads.openwrt.org/snapshots/trunk/ar71xx/packages 
        dest root /opt 
        dest ram /opt/tmp 
        lists_dir ext /opt/tmp/var/opkg-lists 
        EOF 

Let's make sure everything works properly::

        umount /jffs 
        mount -o bind /mnt/sda_part1/root /tmp/root 
        mount -o bind /mnt/sda_part1/opt /opt 
        export LD_LIBRARY_PATH='/opt/lib:/opt/usr/lib:/lib:/usr/lib' 
        opkg update 

You should see:: 

        Downloading http://downloads.openwrt.org/snapshots/trunk/ar71xx/packages/Packages.gz. 
        Inflating http://downloads.openwrt.org/snapshots/trunk/ar71xx/packages/Packages.gz. 
        Updated list of available packages in /opt/tmp/var/opkg-lists/snapshots. 

6. Set the startup script to make the changes take effect each time upon reboot 
-------------------------------------------------------------------------------------

Under DD-WRT’s web interface, Administration->Commands, input the following commands in the window then click "Save Startup":: 

        #!/bin/sh 

        sleep 5 
        if [ -f /mnt/sda_part1/optware.enable ]; then 
        mount -o bind /mnt/sda_part1/etc /etc 
        mount -o bind /mnt/sda_part1/root /tmp/root 
        mount -o bind /mnt/sda_part1/opt /opt 
        else 
        exit 
        fi 

        if [ -d /opt/usr ]; then 
        export LD_LIBRARY_PATH='/opt/lib:/opt/usr/lib:/lib:/usr/lib' 
        export PATH='/opt/bin:/opt/usr/bin:/opt/sbin:/opt/usr/sbin:/bin:/sbin:/usr/sbin:/usr/bin' 
        else 
        exit 
        fi 

Note that some users have reported issues that they were able to fix by making the script sleep 10 seconds instead of 5. 

7. Modification of the profile file 
--------------------------------------------------
SSH into your box then copy/paste the commands below to PuTTY window to create a script running each time when user root logins::

        cat > /mnt/sda_part1/root/.profile << EOF 
        export LD_LIBRARY_PATH='/opt/lib:/opt/usr/lib:/lib:/usr/lib:/opt/usr/local/lib' 
        export PATH='/sbin:/opt/bin:/opt/usr/bin:/opt/sbin:/opt/usr/sbin:/bin:/usr/bin:/usr/sbin:/opt/usr/local/bin' 
        export PS1='\[\033[01;31m\]\u@\h \[\033[01;34m\]\W \$ \[\033[00m\]' 
        export TERMINFO='/opt/usr/share/terminfo' 
        EOF 

The above script will set the variables for us and also provide a nice colored command line prompt. 

8. Reboot and check 
---------------------------------------
Reboot your device with reboot 
When it's back on the track, SSH into your box. 
Install libc with opkg::

        opkg update
        cd /tmp 
        wget http://downloads.openwrt.org/snapshots/trunk/ar71xx/packages/libc_0.9.33.2-1_ar71xx.ipk 
        opkg install libc_0.9.33.2-1_ar71xx.ipk


install transmission
=============================

install
-----------------
::
    
        opkg install transmission-daemon transmission-web 

run transmission and wait 10s for config init::

        transmission-daemon
        killall transmission-daemon

settings.json should be placed at /tmp/mnt/root/.config/transmission-daemon/settings.json


setup config
----------------
make download dir::

        mkdir /mnt/sda_part1/downloads

change download-dir and incomplete-dir in settings.json to your download dir

change rpc-password, rpc-port, rpc-username in settings.json
disable rpc-whitelist for external access to web gui

start transmission-daemon with web-gui::

       export TRANSMISSION_WEB_HOME='/opt/usr/share/transmission/web/'
       transmission-daemon 

visit http://192.168.11.1:rpc-port/

setup startup script
------------------------------------
append following to current startup script::

        if [ -f /mnt/sda_part1/optware.enable ]; then 
        export TRANSMISSION_WEB_HOME='/opt/usr/share/transmission/web/'
        /opt/usr/bin/transmission-daemon -g /tmp/mnt/root/.config/transmission-daemon
        fi  

setup dynamic dns and external web-gui access
-----------------------------------------------
save following command in firewall script(Administration -> Commands)::

    /usr/sbin/iptables -I INPUT 1 -p tcp --dport your-transmission-port -j ACCEPT

create a new domain at any dynamic dns service(ex. freedns.afraid.org) and config the DDNS service in DDWRT webui


