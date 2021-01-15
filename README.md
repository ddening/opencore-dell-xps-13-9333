# opencore-dell-xps-13-9333
A documentation about my progress setting up macOS Catalina on a Dell XPS 13 9333. This is not meant to be a guide. It's rather a collection of certain problems I have encountered during installation and how they got solved. See this just as another source of information for troubleshooting.

## System
![system](resources/catalina.png)

## WiFi / Bluetooth

### Intel Wireless AC 7260
This is the wifi card already builtin in the Dell XPS 13 9333. For your first setup you can use [itlwm](https://openintelwireless.github.io/itlwm/) in combination with [HeliPort](https://openintelwireless.github.io/HeliPort/)
### Broadcom DW1820A
I swapped the original wifi chip for the DW1820A since i was able to get it for a lower price than usual. Keep in mind that the DW1820 and DW180**A** are two different models.

![system](resources/img-01.png)

#### 5GHz connection not working
Out of the box I was not able to connect to any 5GHz network. In order to fix this problem I simply added ```brcmfx-country=XX``` to my ```PciRoot(0x0)/Pci(0x1C,0x2)/Pci(0x0,0x0)``` under DeviceProperties. ```XX = your country code, e.g. US```. To figure out the device ID of your network card you can either use [gfxutil](https://github.com/acidanthera/gfxutil/releases) or [Hackintool](https://github.com/headkaze/Hackintool). Didn't have to make any further adjustments.

#### Bluetooth not working
At some point I encountered a weird bug(?) where I was not able to get any bluetooth connectivity at all. The solution to this problem was even weirder. Booting into my linux system from a USB stick, verifying that bluetooth actually works, and then booting back to macOS solved the bluetooth problem and hasn't occured since. 

## Trackpad
### VoodooPS2 (the easy way)
This is the easy solution to get your trackpad and gestures working. Just use [VoodooPS2 Version 2.1.9](https://github.com/acidanthera/VoodooPS2/releases/tag/2.1.9). As of writing this the Version 2.2.0 was released which didn't seem to work with this particular system.

### VoodooI2C (the non easy way)
Trying to setup VoodooI2C over several days only resulted in the trackpad working with basic features. Gestures and scrolling doesn't seem to function on the Dell XPS 13 9333 this way. The trackpad gets recognised as ```TPD1```, but using VoodooI2C in combination with VoodooI2CHID and remvoing the interrupt sequence as suggested causes the trackpad to run in polling mode only (as seen in the system.log).
Further research might be required at this point. As of now the results with VoodooPS2 are good enough.

### Brightness Keys SSDT Hotpatch
Use the [ACPIdebug.kext](https://github.com/RehabMan/OS-X-ACPI-Debug) to figure out which methods needs to be patched. Refer to the section [Brightness Keys](https://www.tonymacx86.com/threads/guide-patching-dsdt-ssdt-for-laptop-backlight-control.152659/) for more information. Monitor the ```Console.app``` while pressing your key-combinaton to change the brightness. You should get something like this:

IMAGE

The ```system.log``` shows us that we have to patch the methods ```_Q80``` and ```_Q81```. Apply following patch to your ```DSDT.dsl```

```
into method label _Q80 replace_content
begin
// Brightness Up\n
    Notify(\_SB.PCI0.LPCB.PS2K, 0x0406)\n
end;
into method label _Q81 replace_content
begin
// Brightness Down\n
    Notify(\_SB.PCI0.LPCB.PS2K, 0x0405)\n
end;
```
Now in order to create the ```SSDT-XKEY.aml``` we use the tool [Diffmerge](https://sourcegear.com/diffmerge/) to see what changes were made in the ```DSDT.dsl```.
add_screenshot_of_diffmerge
Our final result looks like this.


### Battery SSDT Hotpatch
Before you try to hotpatch your battery I recommend to read this entire guide several times [Battery Status Hotpatch](https://www.tonymacx86.com/threads/guide-using-clover-to-hotpatch-acpi.200137/). The whole process of creating your own ```SSDT``` won't be discussed here. Adding the necessary methods into the right scopes shouldn't be too difficult. I think the only interesting part showing is the final trimmed  ```OperationRegion``` which looks like this. 

```
trimmed
```

This is how the final ```SSDT-XBAT.aml``` looks like.

**Note:** I'm not an expert at hotpatching. This hotpatch might work on my system, but I can't guarantee it will work on yours. Use at your own risk. (You should create your own hotpatch anyways.)


### Audio (ALC668)
The audio device ```ALC3661``` is a rebrand of the ```ALC668```. In order to get the audio working you have you apply the ```layout-id``` mentioned under 
[supported codecs](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs). In my case the ```layout-id=3``` seems to be working fine. If you're trying to get your audio working while using an active ```DSDT.aml``` in your ACPI folder you have to create a SSDT hotpatch instead of using the ```DSDT.aml```. An active ```DSDT.aml``` caused my audio not to function during debugging.



