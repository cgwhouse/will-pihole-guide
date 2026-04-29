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

This is where the Raspberry Pi enters the picture. By the end of this stage, the Pi will be running Pi-hole as the DNS provider for the whole network, with unbound handling recursive lookups behind it.

### Image the SD Card

1. Use `rpi-imager` to flash the SD card. For the OS, choose Alpine Linux (the Raspberry Pi build).

2. Once imaging finishes, copy the "headless" overlay archive (the `.apkovl.tar.gz` file we prepared ahead of time) to the root of the SD card's boot partition. That archive is what tells Alpine to bring up networking and SSH on first boot, so we don't need a keyboard or monitor attached to the Pi.

### First Boot and SSH

1. Insert the SD card into the Pi and connect the Pi to the Deco with an Ethernet cable. Reminder: the cable from the AT&T device goes into the Deco's WAN port; the Pi goes into one of the Deco's LAN ports.

2. Power the Pi on. It may take a couple of minutes to come up. Watch the device list in the Deco settings, and note the IP address it assigns to the Pi.

3. From your computer, SSH into that IP (using a terminal, or PuTTY, or similar). The username is `root`, and no password is required on this first connection.

### Install the Base System

1. Run `setup-alpine`. We'll do a traditional "sys" install to keep things simple.

2. When the script finishes, reboot the Pi. That will drop your SSH session. Once the Pi comes back up, SSH in again — this time using the password you set during `setup-alpine`.

3. Check whether your home network uses IPv6. Not a blocker either way, but a few later steps will depend on the answer.

### Baseline Configuration

Before installing Pi-hole, get the following squared away. Most of these have brief instructions in the Alpine Wiki and/or my edc repo:

- A text editor of your choice
- Enable the community repo, and switch repo URLs to HTTPS
- A shell (I prefer zsh)
- NTP via chrony — apply the small config edit from the Alpine Wiki for good measure
- The `raspberrypi-bootloader-cutdown` package, per the Alpine Wiki
- A firewall (I prefer ufw). Run the prerequisite commands from my edc repo and the Pi-hole docs first, so we don't lock ourselves out
- Replace busybox cron with cronie

Once all of that is in place, reboot the Pi one more time. While you're at it, assign the Pi a static IP via the Deco's settings — Pi-hole needs a stable address.

### Install Pi-hole

1. Make sure bash is installed (the Pi-hole installer needs it), then run the Pi-hole installer script.

2. Open the Pi-hole dashboard in a web browser. Change the auto-generated admin password to something more reasonable.

3. If the dashboard looks healthy, update the Deco's DNS settings to point to the Pi's static IP. On my router this kicks every device off and forces them to reconnect, so expect a brief disruption.

4. Test internet access from a couple of other devices. Everything should work normally, and you should start seeing queries show up on the Pi-hole dashboard.

### Add Unbound

Set up unbound as a recursive resolver in front of Pi-hole, with Cloudflare as the upstream. Use the config snippets from my edc repo as a starting point.
