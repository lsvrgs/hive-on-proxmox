# How to GPU mine cryptocurrency using Hive OS and Proxmox
In order to do this, your motherboard and processor need to support IOMMU, and the feature needs to be enabled in your mobo's BIOS. You'll also need to set up a farm with hiveon.com.

### STEP 1: INSTALL PROXMOX
Flash installation media for Proxmox Virtualization Environment using latest version .ISO at https://proxmox.com/en/downloads/category/iso-images-pve

My tool of choice to do this is Balena Etcher: https://www.balena.io/etcher/

### STEP 2: ENABLE PCI PASSTHROUGH
SSH into your host and run the following command:
>nano /etc/default/grub

Find the line with "GRUB_CMDLINE_LINUX_DEFAULT" and replace it with this: 
>GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on pcie_acs_override=downstream,multifunction video=efifb:eek:ff"

Save and exit, then run:
>update-grub

Reboot.

Now run:
>nano /etc/modules

Add these lines to the file: 
>vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd

Save and exit, then reboot.

Now run:
>nano /etc/modprobe.d/pve-blacklist.conf

Add these lines to the file:
>blacklist nvidia
blacklist radeon
blacklist nouveau

Save and exit, then reboot.

### STEP 3: CREATE VM
Now that you have Proxmox configured for PCI Passthrough, it's time to create a VM in Proxmox. You can use the web GUI for this, so head on over and follow along:
1. Click "Create VM"
2. In the dialog box that appears, make sure you check "Advanced" at the bottom to see extra configuration options.
3. Give your VM a name.
4. Check "Start at boot" to ensure your miner is always up and running then click "Next".
6. Select "Do not use any media" then click "Next".
7. Set System to "q35" then click "Next". 
8. Delete the pre-populated SCSI disk by clicking the red trash can. We'll get to this later. Click "Next".
9. Since we're GPU mining, we can leave everything on the CPU page alone. You only need 1 core. If you're going to CPU mine, you'll need to allocate more. Click "Next".
10. It is recommend to allocate at least 4 GBs of RAM for Hive OS, so enter "4096" and uncheck "Ballooning Device". Click "Next".
11. Select "Intel E1000" as the network device. VirtIO probably works too if you want to leave it at the default settings. 
12. Confirm your selections and click "Finish". Now the fun part.

### STEP 4: DOWNLOAD HIVE OS IMAGE AND EXTRACT TO LOCAL DISK
SSH into your Proxmox server as root and run this command:
>wget https://download.hiveos.farm/history/hiveos-0.6-212-stable@211201.img.xz

Then run:
>xz -dv hiveos-0.6-212-stable@211201.img.xz

Then run: 
>qm importdisk 100 hiveos-0.6-212-stable@211201.img local-lvm 

The above command will vary based on your set up. Make sure to replace 100 with the ID of your new VM and local-lvm with the specific storage type you're using if you decided to configure it differently.

### STEP 5: ATTACH LOCAL DISK TO VM
Head back to the web GUI and select your VM. Select "Hardware" and attach the newly created drive to the VM. Here, you can specify it to be a SATA drive, enable SSD emulation, and enable discard mode. Then you can resize the disk and add on more space. I went with 32 GB (so 25 additional GBs as the expanded installation media is 7 GBs).

### STEP 6: SET YOUR BOOT ORDER AND START VM
Now, make sure your boot order is configured so that the VM knows to read the attached SSD. With all this out of the way, you can click "Start" and see the boot sequence using the Console. 

### STEP 7: SIGN IN TO YOUR RIG
Once the operating system is done booting and running it's first time sequence, you will be prompted to log in to your server. Using the Rig ID and password from the Settings pane of your farm, sign in to your rig. We haven't passed in any GPUs yet, so your farm will be empty; but, this is a necessary step because once we add in the GPUs, you won't be able to access the OS display from the Proxmox console anymore. All of your rig management will happen from Hive's web GUI or VNC. 

### STEP 8: PASS THROUGH YOUR GPUs
This is the fun part! Make sure you stop running this VM, and then click on your VM and under the hardware tab, click "Add" then select "PCI Device". Select the correct device, check "All Functions" then click "Add". Note: when I was doing this, I ran into a GUI bug where I couldn't add more than 5. So, to get around this, you'll have to add the manually using the command line.

SSH into your Proxmox server again (or use an existing connection) and run the following command:
>nano /etc/pve/qemu-server/100.conf

Again, the number of the node will depend on the ID you gave it during set up, so make sure you are using the right one. Now, you will see the configuration file for your VM. Here, you will want to add in the rest of your GPUs using their device signatures as per the GUI. For example:

>hostpci0: 0000:01:00
hostpci1: 0000:02:00
hostpci2: 0000:03:00

The ones you added in the web GUI before hitting an error should be there, so you can use that as a guide. Once you're done, save and exit. 

### STEP 9: PROFIT
That's it! Click "Start" to turn on the VM and let Hive OS boot for 5 minutes before checking your rig management page. Configure your flight sheet and overclocks and you're good to start mining! 

### STEP 10: ???
Now that you are mining, you can use the rest of your system to do other things. The possibilities with proxmox are endless. Thanks for reading! 

#### References
[1] https://pve.proxmox.com/wiki/Pci_passthrough
[2] https://www.youtube.com/watch?v=fgx3NMk6F54
[3] https://www.youtube.com/watch?v=OHiQqJcrjdk

Thank you to these fantastic content creators who got me to where I am today. With the spare resources on my mining rig, I am running a FluxNode. :)

Happy Mining! 

