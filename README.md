# Will's Pi-hole Guide

If you are reading this and you're not Will, know that portions of this advice are specific to his setup. However, you may find some use of it anyway.

If we fail, the most likely reason is that I am a charlatan - in that event, I can suggest other fun uses of your new Raspberry Pi, or otherwise I will buy it from you to recoup the loss.

As we're going through this, if something doesn't seem right / stops working, don't move on - we should be able to undo whatever we've done so far, and troubleshoot further. Go slow!

## Stage 1 - Prologue

The steps in this stage will not involve the Raspberry Pi at all. In fact, they are things you probably want to do anyway; you will almost certainly notice increased performance on your home network.

The main goal here is to streamline the network itself.

### Disable AT&T Wi-Fi

We don't need the AT&T device to also output a Wi-Fi signal, since we have the Deco for that.

1. Make it so the AT&T device only has two things plugged into it (not counting a power cable):
   - The coax cable / whatever cable carries the signal from the wall
   - An ethernet cable connecting to the main Deco router. This cable should be plugged into "Port 1" (5Gbps, may be indicated by a blue line?) on the AT&T side, and "WAN" on the Deco side.

2. If anything had to be moved to accomplish the above, Make sure it's still working.

3. Somewhere on the AT&T device will be a "Device Access Code" - write that down.

4. Temporarily, connect a computer via ethernet directly into the AT&T device using one of the other ethernet ports on the back.

5. Open a web browser on that computer, and navigate via web browser to `http://192.168.1.254`. I am not 100% sure that is the right IP address, we'll figure out the right one if it isn't. This should take us to AT&T settings.

6. Navigate to Home Network -> Wi-Fi, then click on Advanced Options.

7. Change the "Wi-Fi Operation" for both 2.4 GHz and 5 GHz bands, to "Off". Save this change, which will probably require Device Access Code.

8. Note that any devices which were using the direct AT&T Wi-Fi, will now need to be pointed at the Deco instead. This is another spot that we stop and be sure everything is working again.

### IP Passthrough

Bridge the connection from the wall / AT&T, to the Deco. After this is complete, the AT&T device will only have one job that it does really well: make the internet connection from your ISP available to your home network.

1. Back in AT&T settings, navigate to Firewall -> IP Passthrough.

2. Set "Allocation Mode" to Passthrough, and set "Passthrough Mode" to DHCPS-fixed.

3. Under "Passthrough Fixed MAC Address" there should be a drop-down list where we can look for the Deco router. If it doesn't show up, we can select "Manual" and type in the WAN MAC address of the Deco router, which we should be able to find somewhere in those settings.

4. "Passthrough DHCP Lease" is probably fine to leave alone / whatever the default is.

5. Save these updates too.

### Disable AT&T Firewall

We don't need the AT&T device to do anything except give internet to the Deco, and "anything" also includes firewall stuff.

1. Navigate to Firewall -> Packet Filter.

2. Click "Disable Packet Filters".

3. Navigate to Firewall -> Firewall Advanced and turn off all the toggles on this page.

4. Click "Save".

### Wrapping Up Stage 1

1. Turn off both the Deco and the AT&T device. Make sure that the ethernet from the AT&T's "Port 1" is plugged into the WAN port of the Deco. Unplug the extra ethernet we used to change these settings.

2. Turn on the AT&T device again (needs to have been off for at least 30 seconds) and wait until the front light turns solid white.

3. Turn the Deco back on, wait for it to boot up completely / for the lights to stop changing, then make sure everything still works.

## Stage 2 - Core Pi-hole Setup

1. Image the SD card using `rpi-imager`. When selecting the OS, choose Alpine Linux.

2. After the imager does its thing, put the "headless" archive thing at the root of the SD card.

3. Put SD card into the Pi, and connect it to the router via Ethernet cable. Reminder: the Ethernet connecting the modem to the router should be plugged into the WAN port, but the router should have multiple LAN ports available as well (they are all Ethernet ports, but they serve different purposes). You want the Pi connected to a LAN port.

4. The Pi should boot up, it may take a couple minutes. Wait until you can see it as a device on the network in your router settings. Note the IP address your router gave it.

5. Using a terminal on your computer (or some other dedicated program, e.g. PuTTY), ssh into the Pi using the IP address you found in router settings. The username will be "root" and there should be no password required.

6. Run the `setup-alpine` script to install the base system. To keep things simple, we will be doing a traditional "sys" install.

7. Once the script completes, reboot the Pi which should kick you out of your ssh session. Once it comes back up, ssh in again (will need to provide a password this time).

8. Figure out whether your home network uses IPv6. No big deal either way, but it will affect some of the steps we perform later on.

9. Establish the following baseline:
   - Text editor
   - Enable community repo + use https
   - Shell (I prefer zsh)
   - NTP / chrony (config edit from Alpine Wiki for good measure)
   - `raspberrypi-bootloader-cutdown` thing from Alpine Wiki
   - Firewall (I prefer ufw), can go ahead and do the prerequisite commands from Cristian's edc repo + the Pi-hole docs
   - Replace busybox cron with cronie

10. You probably want to throw one more reboot in after all that configuring is done. Also, go ahead and give the Pi a static IP address via router settings.

11. After the dust settles from all of the above, ensure bash is installed and then run the Pi-hole setup script.

12. Access the dashboard via web browser, remembering to change the auto-generated password to something more reasonable.

13. If everything looks reasonable, edit the router settings so that the Pi is now the DNS provider for your home network. On my router, this causes everything to get booted off and reconnect.

14. Test internet access on some other device(s), everything should be working as normal and we should see requests coming through on the Pi-hole dashboard.

15. Setup unbound, using Cloudflare as the upstream, take config snippets from edc repo.
