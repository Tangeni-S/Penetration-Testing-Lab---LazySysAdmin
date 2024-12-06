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
Analyzing the content in this location, I find a password in the deets.txt file
- Password: **12345**
![deets file](https://github.com/user-attachments/assets/0969e346-07e5-4a37-a059-dcb1e1fb1971)

**Webpage Browsing**
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

The Themes window loads, it shows the current theme that is active as is underlined, in this case **twenty seventeen**. To make changes to this theme we select Editor as indicated

![10](https://github.com/user-attachments/assets/f044b216-45bf-4873-a4c9-e1dcb628be8e)

With the editor open , I will select the **404 Template** to edit for this them. I replaced the content indicated by the brace with  the content of the **php-reverse-shell**   script
found at **/usr/share/webshells/php/php-reverse-shell.php** 

![12](https://github.com/user-attachments/assets/74f69943-0afa-4eba-b816-198e5cb3b0c3) 

Having replaced the default content with the reverse shell script i scroll down to change the listening IP address to my kali machine IP address 

![13](https://github.com/user-attachments/assets/ab6de75f-cfb3-45c2-b8f4-28fcea93492f)

Having changed my IP , press the Update File button and ensure it has been update successfully.

![14](https://github.com/user-attachments/assets/6fb5f092-77eb-4f9b-b996-ca2f4831ca8d) 

With that set, i proceed to go to my kali and start a listener with Netcat listening on the specified port , **1234**.

![15](https://github.com/user-attachments/assets/c4c6f816-32d8-4236-98ce-9fc94ee134d2)

I proceed to visit the 404 template page in order to trigger the reverse shell connection. The page being at 
**http://15.10.1.128/wordpress/wp-content/themes/twentyseventeen/404.php**. It is shown in the SMB share location below:

![16](https://github.com/user-attachments/assets/46481ed3-2835-45d5-aaa6-d964eddfdfa5)

Entering the following URL in my browser: 

![17](https://github.com/user-attachments/assets/31e502a3-825d-4c18-a67c-6c2863932aee)

Presented me with the reverse shell from the listener started previously on port 1234:

![18](https://github.com/user-attachments/assets/a22f3839-90f9-4f77-8ec4-ddb95cef528b)

I proceed to spawn an interactive shell using python with the following command:
**python -c "import pty;pty.spawn('/bin/bash')"**

![19](https://github.com/user-attachments/assets/1b81aeb2-636e-44d4-b773-8574ea46bc7b)

Take not in a error in format will result in the python shell not spawning as indicated by the X. 

With the python shell spawned ,  list the directory i am currently in and cd into home to view name of the user I am currently logged in as. 
The user being: **togie**

![20](https://github.com/user-attachments/assets/1cf9396b-2d81-41a2-926b-5c70aa890ee2)


I **cd** into togie and **ls -al** to view all contents to see if i find the flag here, but it is not. 
I assume since it is not here i can assume it is in the **root** directory


![21](https://github.com/user-attachments/assets/100c9c4d-5e2e-4364-a5c6-bdf34963ac81)

Trying to access the root directory results in Access denied. This means tobie lacks the required permissions to access root directory.

**Going back to the Password found in the deets.txt file**
I attempt to use that password to login to the user account , togie, over ssh since we know from the service there is ssh running and we have a user account name but a unknown password.. 
I open a new terminal window and do so accordingly:

![23](https://github.com/user-attachments/assets/43fd3c29-707c-4268-aa3d-838cbda26d22)

Login Successful, the password 12345 found earlier was for togie.

## Alternatively 
Whilst in the window where i spawned the python  shell , i can use switch user command to switch to the togie user account

Comamand: **su togie** >> Password: **12345**
![24](https://github.com/user-attachments/assets/1dbe86a2-3dcb-4ac6-a270-f0ef1c4f451d)



## Privilege Escalation

Logged in as Togie, he still cannot access root directory
![25](https://github.com/user-attachments/assets/935b0219-a631-432b-8a84-303ea5bc38f6) 


I proceed to view what commands Togie can run using sudo by running the following command: 
**sudo -l**

![26](https://github.com/user-attachments/assets/a25f73e5-560d-4cc3-8085-a5aa94f7d76b)

It shows that togie can run any command with sudo. With that knowledge we can attempt to switch to the root use by running the command:

Command: ** sudo su **

![root_achieved](https://github.com/user-attachments/assets/4998d505-9dec-4751-aae1-ad518229f8f7)

Command ran successfully and we have acquired root access to the server. We should  now be able to proceed to the root directory and view its contents:

![27](https://github.com/user-attachments/assets/f1781507-88ae-4192-8f06-5429c0e20b27)


## CONTENTS OF FLAG

![flag](https://github.com/user-attachments/assets/9f9d835b-ecee-4612-8273-7aeec38e1b80)
















