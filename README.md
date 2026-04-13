# Will's Pi-hole Guide

If you are reading this and you're not Will, know that portions of this advice are specific to his setup. However, you may find some use of it anyway.

If we fail, the most likely reason is that I am a charlatan - in that event, I can suggest other fun uses of your new Raspberry Pi, or otherwise I will buy it from you to recoup the loss.

As we're going through this, if something doesn't seem right / stops working, don't move on - we should be able to undo whatever we've done so far, and troubleshoot further. Go slow!

## Stage 1 - Prologue

The steps in this stage will not involve the Raspberry Pi at all. In fact, they are things you probably want to do anyway; you will almost certainly notice increased performance on your home network.

The main goal here is to streamline the network itself.

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

9. Back in AT&T settings, navigate to Firewall -> IP Passthrough.

10. Set "Allocation Mode" to Passthrough, and set "Passthrough Mode" to DHCPS-fixed.

11. Under "Passthrough Fixed MAC Address" there should be a drop-down list where we can look for the Deco router. If it doesn't show up, we can select "Manual" and type in the WAN MAC address of the Deco router, which we should be able to find somewhere in those settings.

12. "Passthrough DHCP Lease" is probably fine to leave alone / whatever the default is.

13. Save these updates too.

14. Navigate to Firewall -> Packet Filter.

15. Click "Disable Packet Filters".

16. Navigate to Firewall -> Firewall Advanced and turn off all the toggles on this page.

17. Click "Save".

18. Turn off both the Deco and the AT&T device. Make sure that the ethernet from the AT&T's "Port 1" is plugged into the WAN port of the Deco. Unplug the extra ethernet we used to change these settings.

19. Turn on the AT&T device again (needs to have been off for at least 30 seconds) and wait until the front light turns solid white.

20. Turn the Deco back on, wait for it to boot up completely / for the lights to stop changing, then make sure everything still works.
