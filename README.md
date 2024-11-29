# Penetration-Testing-Lab---LazySysAdmin

## Objective
This lab is aimed to test my penetration testing abilities in a controlled enviroment. The primary focus is to simulate to as close as possible a live penetration test according to the CompTIA Pentest+ methodology
which entails the following steps:
- Step 1: Planning and Scoping
- Step 2: Information Gathering and Vulnerability Scanning
- Step 3: Attacks and Exploits
- Step 4: Reporting and Communication


### Requirements
This lab will require me to have the following in order to attempt:
  1) Vmware Workstation (This is my preferred Hypervisor of choice, you can as well utilize VirtualBox)
  2) Kali Linux (I utilize the preconfigured Kali images found at [osboxes.org](https://www.osboxes.org/vmware-images/) to save time on having to do a complete install)
  3) LazySysAdmin VM found at vulnhub.com at the following link >>> https://www.vulnhub.com/entry/lazysysadmin-1,205/



### Tools Used
-Nmap

### Skills Learned


### Procedure
#### 1) Setting up the VM's
Starting off from having downloaded  the image files for Kali and the vulnerable machine (LazySysAdmin), and loading them into VMware, 
I configure the RAM to my desired amounts as to not drastically affect my host's performance. 

![Capture](https://github.com/user-attachments/assets/0ef5cc73-3633-40d8-a2da-9368a1fd3c68)  ![Capture JPG2](https://github.com/user-attachments/assets/9cc1ac30-35d9-4aa8-80aa-d9c1d82c763b)

After that I proceed to configure the network settings which is crucial. In my case I want my VM's to be configured to Host Only but with my own specified IP network. I do so by opening the Virtual Network Editor in VMware

![Capture JPG3](https://github.com/user-attachments/assets/b07c9e81-0be5-4928-8584-ec7bc815957b) Selecting the Change Settings button allows me to adjust the configurations


Select Add Network and specify  the Virtual Network interface to utilize. In my case i select vmnet5. Set it to host-only and proceed to assign the network address of your choice accordingly. I utilize 15.10.1.0 /24 network.

![Capture JPG4](https://github.com/user-attachments/assets/1bd086cc-7b52-45d8-a4de-5de296e39f5b) 

After applying the changes , close the dialog by selecting ok. Proceed to configure the network settings of each VM to be set to VMnet5 as follows

![Capture JPG5](https://github.com/user-attachments/assets/a89e3882-d77d-470f-9f64-d0c0e296f715)
![Capture JPG6](https://github.com/user-attachments/assets/e1065266-a58d-4985-a418-669aa19ec553)

With the VM's configured, the fun part begins!! :) 


