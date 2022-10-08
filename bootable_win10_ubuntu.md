# Create a bootable windows 10 usb in ubuntu

If you want to reinstall Windows 10 completely on a device or you simply want to have a Windows installation disk ready, you will need a bootable Windows 10 USB.

## Prerequisite
* Download a Microsoft Windows 10 ISO from the website [here](https://www.microsoft.com/en-in/software-download/windows10ISO).\
Note: the download link is only valid for 24 hours.
* A USB of at least 8 GB.

There are 2 ways to create a bootable windows 10 usb stick

## Create bootable disk using ubuntu 
This method of mounting the ISO image of Windows to a USB disk formatted in ExFAT system is the easiest, although there could be instances where it wouldn’t boot.

### Format usb stick:
Format your usb using the disks tool on ubuntu.
* select your USB stick and format.
* it will ask for a partitioning scheme. Select GPT and hit format. Note: if a warning will pop up that your data will be erased, simply proceed.
* create a partition on the newly formatted USB by clicking the (+) button.
* input the partition size and click next.
* name your usb and create.

### Copy content of iso to usb:
copy the content of the Windows 10 ISO to the USB.

because the max size for exFAT files is 4GB, you will have some issues copying the files of the iso to the USB.

to solve this:
* open the ISO by right click and 'open with the other application'.
* create a folder on your local device and copy the content of the iso file into the folder.
* zip and compress the folder
* copy the zippped folder into the usb stick.
* in the USB stick, unzip the folder in the root directory of the usb.
* when complete, you can delete the zipped folder.
* congrats you know have a bootable usb stick.

## Create bootable disk using ventoy
The second method is to use a tool like Ventoy. It creates a UEFI compatible bootable disk.

### Download and install ventoy:
Ventoy is a combo of GUI and CLI tool. It can be used on any Linux distribution. Download Ventoy for Linux from the release page of its GitHub repository [here](https://github.com/ventoy/Ventoy/releases)

* create a folder and extract the zipped ventoy file into it.
* the extracted folder, and you’ll find a few scripts in it.
* use the commandline to run the file called VentoyGUI. x86_64. 
```
sudo ./VentoyGUI. x86_64`
```

### Use ventoy to create bootable windows 10 usb:
* Ventoy finds your usb stick and has the option to create a bootable disk with secure boot. go for a UEFI installation and select GPT for partitioning scheme.
* once set, hit install.
* ventoy creates an exFAT partition where you can copy the ISO file.
* copy and paste the iso image
* once finshed, eject the drive and WAIT until the prompts you to remove the drive. if you do not wait, the drive will miss out on some nessasry drivers.

## Use the bootable windowns 10 usb
Now our bootable USB stick is ready,
plug into the device you wish to install windows.
* start and go to the BIOS with F2/F10/F12.
* go to secure boot option and disable it.
* go to boot order and boot from UEFI USB stick.
* you can proceed from here.

