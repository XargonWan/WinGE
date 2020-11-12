## Introduction

My PC is a 4 year old laptop and nowadays cannot handle the newest game very smoothly even in a medium/low graphics.
At least it's what I thought: on my main computer I am used to install not only gaming related things but I use it for almost every purpose, so I thought to do a second boot to install a clean copy of Windows 10 specifically to run games and nothing else, that it's what I called Windows Gaming Edition (WinGE in short).
However I didn't feel like to create another partition because the space on my disk is not a lot, I wished something that I can quickly backup and that's dynamic, so I had the idea: "I will create a dynamic virtual disk".


## Why a dynamic virtual disk (vhdx)?

### PROS:
- Dynamic: means that if that fake partition is 40GB it will be used 40GB only, a full partition takes up all the space you partition of course.
- Easy backupable and restorable: that virtual disk is a .vhdx file placed on my C drive, and I can easly copy it on an external drive and restore it by copying it back if needed. True: it's a big file (mine is 50GB now) but also true that if I create a disk image is not less.
- I can easy launch the .vhdx as a Virtual Machine in order to update it and edit it...

### CONS:
- ...but the Windows Feature Updates are not installable on a virtual disk, so I MUST install them via Hyper-V (well, once every... 6 moths?).
- Because is dynamic is easy do be damaged if there is a blackout or such, but who cares: I got a backup in my external disk, so just a copy-paste is enough to restore it.


## Windows Gaming Edition Features:
- Quick boot time: only the essential things are installed: Playnite, Discord, and all the stores like Steam and such.
- Console feeling: you're directly into the Playnite games enviornment like you're playing a console.
- Full hardware power for gaming: there are no software distracting you from the gaming.
- Playnite is synced: Playnite is synced with my main partition as, like the games, it's installed on the common D: drive.
- [OPTIONAL] Reboot directly into WinGE skipping the bootloader.
- [EXPERIMENTAL] Playnite as shell: Explorer doesn;t even start, even less boot time and more continuity. (This option however prevents the startup apps to start, like Discord, and some windows feature to be used, like the keyboard volume control, I need to find out how to address this issues).

This Gaming Edition is really giving a second wind to my PC, I can play the latest games without lag on a 5 year old notebook with a very nice graphics.


## Needed tools:
- Any Windows 10 installation iso.
- Hyper-V usually it's not coming with Windows Home Edition but there's a trick to install it even there. (You can find the file attached).
- Fix for the VHD "no space" issue (You can find the file attached).
- At least 50GB of disk space (SSD partition duggested).
- You should know if your PC has got a BIOS or an UEFI (check your motherboard on the internet).

## Guide:

**DISCLAIMER:** I did this guide one month ago and I am writing it out from my memory and my scraps, please tell me if I miss something.
I take no responsability if you mess up something, if you decide to follow my guide is all up to you.

### 1. Creating the VHDX
Open cmd as administrator (WIN+R, cmd, Open as administrator) and give the following commands:
```
diskpart
create vdisk file="C:\WinGE.vhdx" maximum=102400 type=expandable
attach vdisk
clean
```

Now if you have BIOS please follow 1.1, if yo have UEFI follow 1.2.

#### 1.1 BIOS
```
create partition primary size=100
format quick fs=ntfs label="System"
assign letter="S"
active
create partition primary
format quick fs=ntfs label="Main"
assign letter="M"
exit
```

#### 1.2 UEFI
```
convert gpt
create partition efi size=100
format quick fs=fat32 label="System"
assign letter="S"
create partition msr size=128
create partition primary 
format quick fs=ntfs label="Main"
assign letter="M"
exit
```

### 2. Installing Hyper-V:
Launch Hyper-V-Enabler.bat and press Y and Enter if needed.
This will take a while, lay back and have a tea.

### 3. Hyper-V procedures
#### 3.1 Hyper-V inital setup:
- WIN button, write Hyper and you should see the Hyper-V console, open it.
- On the rigth panel press "Manage Virtual Commuters" (or similar).
- Select "New Virtual Commuter".
- Give it a name, I used "Ethernet".
- Connection type: External network.
- Select you network card and check the flag under it (Allow sharing...)

#### 3.2 Creating a new Virtual Machine:
- On the right panel: New > Virtual Machine.
- Next, Input "Windows Gaming Edition" as name.
- Choose Generation 1 if you have a normal BIOS or Generation 2 if you have an UEFI. Next.
- Set at least 2048 in memry field. Next.
- Select the Connection that we created earlier (I called it Ethernet). Next.
- Now press the second check: Use an existing virtual drive, browse, and select your WinGE.hdx that we have created previously. Next.
- Second check: "Install a operative system from a disk image" and browse for the Windows 10 installer iso (not for the WinGE.vhdx file!).
- End.

#### 3.3 Setting up Windows Gaming Edition:
- Install Windows 10 as you would normally do, format the disk if necessary, don;t worry it's a virtual disk and nothing can happen to your real one.
- Do the needed Windows updates.
- Run the "VHD Expansion Fix.reg" file, otherwise you will have issues when you will really boot your system.
- Install Playnite.
- Install what you like but keep in mind to install the least as possible to don't consume any resources and avoid heavy softwares like Chrome. I just installed Discord, Steam and the other stores such as GOG, UBI Connect, Origin, Bethesda, itch.io, Twitch and Battle.net.

### 4. Tweaking WinGE for maximum performance:
- First uninstall everything you don't need from Windows 10 (aka bloatware) by going on install/remove programs on the Control Panel.
- Go here: https://thegeekpage.com/optimize-your-windows-10-pc-for-gaming/
and follow the setps 1, 2 and 3, maybe the other are good as well but I didn't try them, I don't know if they're messing up with the virtual drive.
- Then follow the step 4 of the previous guide but until the point 8 only (again, I don't know if the others are good, I didn't try them).
- Shutdown the WindowsGE inside the virtual machine (don't just close the VM, press Start and then Shutdown!).

### 5. Add WinGE to the boot manager:
Now that everything is done in the VM, it's time to boot it natively, but first we have to tell to the windows bootloader to add it to the list, the second part is different if you're on a BIOS or an UEFI, so just check the desired paragraph.
As already done before open cmd as administrator.

```
diskpart
select vdisk file=C:\WinGE.vhdx
attach vdisk
list volume
```

Now check the volume number of your WinGE.vhdx, let's suppose that it's the number 3:

```
select volume 3
assign letter=v
exit
```

#### 5.1 BIOS
```
V:
cd v:\windows\system32
bcdboot v:\windows /s S: /f BIOS
diskpart
select vdisk file=C:\WinGE.vhdx
detach vdisk
exit
```

#### 5.2 UEFI
```
V:\
cd v:\windows\system32
bcdboot v:\windows /s S: /f UEFI
diskpart
select vdisk file=C:\WinGE.vhdx
detach vdisk
exit
```

Now it's time to reboot and select the newly boot entry, to know which one is the Gaming Edition just check the one that carries the WinGE.vhdx indication.


### 6. Drives letter check
If, like me, you;re using a second drive/partition to store your games and playnite you may notice that the drive that was for example D: into your main partiton now is different, in my case was E; and of course all the Playnite paths are becoming not correct because they're pointing to D:\Games (in my case).
In order to fix this we have to:
- open the regedit (WIN+R, regedit, enter).
- browse to HKEY_LOCAL_MACHINE\SYSTEM\MountedDevices
- Change the drive letter, in my case for example:
\DosDevices\D: -> \DosDevices\Q:
\DosDevices\E: -> \DosDevices\D:
\DosDevices\Q: -> \DosDevices\E:
So in this way I swapped D: and E: devices
- Reboot to apply the changes

### 7. Install the missing drivers
Try to not install the driver's applications if not needed, just keep at the minimum, I just installed Nviidia GeForce experience and that's all.

### 8. Setup Playnite
In my case I wished to make that the Playnite on WinGE is the same of my main partition, so I had to correctly link the database location.
- Open Playnite
- Settings (F4)
- Advanced
- Database Location: browse your Playnite folder on the D: partition
- Close and reopen Playnite, et voila: you get everything back.
- Now you an do the needed setups like linking the accounts and libraries.
- Set Playnite at the startup with the FullScreen mode directly enabled.

### 9. Startup programs
Now that everything is set it's time to add the desired programs ot the startup, I added Playnite and Discord for example.
Remember to run the least as possibile.

### 10. [OPTIONAL] Tweaking the UI
Now that everything is set up you can give a little graphial tweaking that won'impact on the performances sich as:
- Remove the wallpaper and put a monochrome color, I used a blue tone taken from the default Playnite theme.
- Make the Windows bar as smallet as possible and acitvate the auto hide.
- Hide desktop icons (so when Playnite will start you will not see them and that gives a sense of continuity having only the Playnite splash screen over the previous mentioned dark blue, very elegant.
- Change cursors: I searched on the internet some orange cursors thate are very good with Playnite and I choose these: http://www.rw-designer.com/cursor-set/delta-neon-orange
I may pack everything in a theme if you're interested.

### 11. [OPTIONAL] Set Playnite as Windows Shell
If you wish to skip completely the Windows Explorer loading times and have only Playnite you can put the Playnite.FullscreenApp.exe as windows sell, in order to do that:
- open regedit
- navigate to HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\
- Edit the Shell entry replacing explorer with the full patch of Playnite such as C:\Users\Username\AppData\Local\Playnite\Playnite.FullscreenApp.exe

Beware that this is disabling all the windows hotkeys such as volume +, volume -, WIN.
In order to launch any other app you need to press CTRL+ALT+DEL, Task Manager, File -> Run (or add it as a software inside Playnite).
I din't find any workarounds to enable them yet, but I may update the guide as soon as I find it.

### 12. [OPTIONAL] Quick reboot link
If you want to boot from your main partition directly into the Windows Gaming Edition you can reate a batch file as follows:
```
bcdedit /bootsequence {7feq7c4g-219q-19hu-7c14-47349d3f15z4} /addfirst
Shutdown /r /t 0
```
where {7feq7c4g-219q-19hu-7c14-47349d3f15z4} must be replaced withthe ID of your WinGE, in order to find it open the prompt as admin and do:
```
bcdedit
```
and find the rows
'''
identifier          {7feq7c4g-219q-19hu-7c14-47349d3f15z4}
device              vhd=[C:]\WinGE.vhdx
'''
Then just insert your code as I did in the batch file.


## FAQ(s):

- Is it easy to create this?
Not really, you should be a bit expert in operative systems, but I will try to make the guide as most easy as possible.

- Can't you just packetize and share an installer or a configurator?
Maybe it will be possible to create a custom Windows 10 installer, but I have no experience in this, if someone wants to help me ti will be very welcome.

- Will you add any picture to the guide?
I don't have any plans to do all this again, so no: but feel free to share yours and I will add them.

- Don't you use an anitvirus?
Nope, Windows Defender is enough nowdays, and any other antivirus will decrease your gaming performance.

- Are we creating a Virtual Machine? It's not a good idea to play on a virtual machine.
I know, trust me, I'm The Doctor: we are using a Virtual Machine only for the initial setup and update purposes.


## Final notes
Please feel free to improve and suggest!
And if you want to cooperate, like into automating this process feel free to do it.
