## Introduction
I owned an Asus RT-AC86U and I use it as a printer server. Starting from firmware version 3.0.0.4.386, I started having issue where clients are unable to print after the printer was powered off for a certain duration. Once this happens, I have to restart the printer server via the WebUI. I reached out to Asus but the issue is never fixed and the technical support seems to be unable to replicate it on their side.

This is a workaround to automate the printer server restart whenever a printer is connected (or powered on) to the router by hooking into the events in ```hotplug2```.

## What does ```init_start``` do?
1. Performs a md5 check on ```/rom/etc/hotplug2.rules``` using ```/jffs/custom/hotplug2.rules.md5```.
2. If failed or ```/jffs/custom/hotplug2.rules.md5``` is missing,
   * A copy of  ```/rom/etc/hotplug2.rules``` as ```/jffs/custom/hotplug2.rules```is made and ```/jffs/custom/hotplug2.rules.md5``` is generated.
   * A line is added to execute ```restart_printer```.
3. Check if ```/etc/hotplug2.rules``` is symbolic linked to ```/jffs/custom/hotplug2.rules```.
4. If not, replace the the symbolic link and restart ```hotplug2```.

## Installation
1. Copy the scripts in ```/jffs/scripts``` into the same path in your router.
2. Register ```init_start``` to be executed on ```usbmount``` by executing the following command:
    ```
    nvram set script_usbmount=/jffs/scripts/init-start
    nvram commit
    ```
3. Reboot the router.
4. You may need to perform step 2 again whenever you upgrade the firmware.

Note: Since we're using ```script_usbmount``` to initiate the script, you will need to permanently have a usb drive plugged into one of the USB port of the router.

## Verify that the scripts are working
1. Browse to your router WebUI.
2. Go to ```System Log```.
3. Turn on (or off) your printer.
4. You should see the following events in the log.
    ```
    [timestamp] rc_service: service xxxx:notify_rc restart_lpd
    [timestamp] rc_service: service xxxx:notify_rc restart_u2ec
    ```
