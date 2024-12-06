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
(**Disclaimer: This VM was created by the user, Togie Mcdogie and I have no claim/credit to any part of its development**)



### Tools Used
- Nmap 
- Wappalyzer
- 


### Skills Learned / Developed
1) Passive reconnaissance of a network subnet & devices
2) Directory enumeration of a hosted website
3) 

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


#### 2) Information Gathering and Vulnerability Scanning

Having both VM's running I proceed to do a basic scan of the subnet network in order to identify live hosts.  This results in 1 host being detected with a number of active ports discovered 

SCAN: nmap 15.10.1.128
![Capture JPG7](https://github.com/user-attachments/assets/78929a3c-7ce6-48b6-b04e-f52a5e85f50e)

Proceed to gather further details of the host ports. I run a nmap scan for Service detection(-sV) to get possible details on services running on the ports as well as operating system detection(-O)

SCAN: nmap -sV -O 15.10.1.128 
![Capture JPG8](https://github.com/user-attachments/assets/fe6d404e-ed60-463e-ac75-961fbbed2cb2)

The above information helps me deduce that there is an Apache Web Server (Port 80) running on this machine and as well an mysql server(Port 3306). I note these as major points of interest.
- SMB ports 139 and 445
- HTTP Web Server on Port 80
- mysql Server port 3306
  
Ideally i wanted to look more into the ports running Samba. Samba is an open source software that allows the sharing of file and print services through the ports discovered. 

![1](https://github.com/user-attachments/assets/4a899a4e-adda-4656-a763-f5fe9b5f1589)
**nmap 15.10.1.128 --script smb-enum-shares**

I run another nmap scan with a script to enumerate the smb shares on the server and it returns promising results. From the above, the most promising share would be the share$
because the path for the share gives an indication of the directory for a website being hosted which we could confirm due to the previous nmap scan showing an Apache web server
running on **port 80**.

![2](https://github.com/user-attachments/assets/8eacfaa6-576a-4c90-b8da-ae1d502e67b6)
**smbclient '\\15.10.1.128\share$'**

I proceed to connect to the share and list the directory contents. The content confirms that this is the directory for a website. 

https://github.com/user-attachments/assets/5f0dd0b7-bbfa-411f-8b0b-d1558a041db9

Browsing the site, not much interactive content found.  

Running a scan with nikto I confirm that the directories are for this website with additional other information 

![3](https://github.com/user-attachments/assets/f027ae18-94cd-4841-99fd-0b765945f350)

This shows that the website uses wordpress and there is a wordpress login page that we can look to access in order to gain management access. 
Whilst trying common login credentials on the page , i managed to confirm that there is an account by the name of admin existing as shown below..

![4](https://github.com/user-attachments/assets/b3eed7b8-6630-49f4-bf0b-90afbaca54ad)

The error message confirms for me that **admin** username exists though i got the wrong password.

I proceed to analyze the SMB share further by navigating into the wordpress directory

![6](https://github.com/user-attachments/assets/3598a4d8-bb2a-45e9-b32c-2c2f7962e50d)

Having analysed the files in the directory , the **wp-config.php** i identify contains the credentials for the wordpress login page  as shown below

![7](https://github.com/user-attachments/assets/974f2309-2040-49bf-8bbc-1bec08051998)

Username: Admin

Password: TogieMYSQL12345^^

Using these credentials, the login was successfull on the Wordpress login page bringing me to the Admin page.

![8](https://github.com/user-attachments/assets/76d2243b-7824-4d45-b135-e7551acc253b)

From this point, I seek to find a means of gaining a shell to the server by changing the contents of a certain php file that is used by the website.
I do so by hovering over appearance and selecting themes. 

![9](https://github.com/user-attachments/assets/2b30ad1a-a796-40c6-8fa7-1339c0177a1c)

The Themes window loads, it shows the current theme that is active as is underlined. To make changes to this theme we select Editor as indicated

![10](https://github.com/user-attachments/assets/f044b216-45bf-4873-a4c9-e1dcb628be8e)









