# HomeLab-1-Cloud-based-Storage-Server
## Introduction
In this project, I am going to turn my unused old laptop into a Cloud-based storage server and try to connect it to the WAN so that it can be accessible remotely with a different WiFi. The sake of this project is to learn the OS, Virtualization, Computer Architecture and Networking fundamental. In this project, we will be installing NextCloud as my Cloud-based storage server's platform. Next, I am going to treat the laptop as a newly bought machine just like those newly arrived uninstalled servers in data centers. Therefore, I will be performing deep formatting, basic Windows OS' installation, virtualized OS (WSL)'s installation, NextCloud Cloud Server Platform's installation, LAN routing and WAN routing.
## Steps and Methodologies
### 1. Laptop Deep Clean & Windows OS Installation.
<p align="center">
  <img width="60%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/image.png?raw=true">
  <br> Figure 1: Disk Deep Clean
</p>
<p align="center">
  <img width="60%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/Untitled.png?raw=true">
  <br> Figure 2: Windows OS Installation
</p>
Go to CMD, Diskpart command and perform the necessary commands to clean the disk, and perform partitioning on the disk. After that, proceed with the Windows Installation steps.

### 2. Virtualization and Windows Subsystem Linux (WSL) Installation
**Achitecture Design**
<p align="center">
  <img width="60%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/image1.png?raw=true">
  <br> Figure 3: OS & Virtualization Architecture Design
</p>
I. Enable WSL and Virtual Machine Platform

``` Shell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all
```

<p align="center">
  <img width="60%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/Screenshot%202024-09-16%20231436.png?raw=true">
  <br> Figure 4: Enabling Virtualization Platform
</p>

II. Enable WSL 2

``` Shell
# enable virtualization platform
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
# enable wsl2
wsl --set-default-version 2
# download the wsl kernel update
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi -OutFile .\wsl_update_x64.msi
# reset progress preference
$ProgressPreference = 'Continue'
# install the downloaded file
.\wsl_update_x64.msi
```

III. Install WSL
``` Shell
wsl --list --online
wsl --install -d Ubuntu
sudo apt update
sudo apt upgrade -y
```
Pre-requisite Packages:
``` Shell
# zip install
sudo apt install unzip wget -y
# install Apache HTTPD and MySQL
sudo apt install apache2 mariadb-server mariadb-client -y
# install PHP components
sudo apt install php libapache2-mod-php php-mysql php-common php-cli php-common php-json php-opcache php-readline php-intl php-json php-gd php-mbstring php-mysql php-xml php-zip php-curl -y
# start the mariadb service
sudo service mariadb start
# configure the MySQL database
sudo su
mysql_secure_installation
```
### 3. Creating an SQL Database
``` Shell
mysql -u root -p
```
``` Shell
SHOW DATABASES; # if database created, no need to create

CREATE DATABASE clouddb;
GRANT ALL ON clouddb.* to 'user'@'localhost' IDENTIFIED BY 'Password_01';
FLUSH PRIVILEGES;
EXIT;
exit
```
### 4. Install NextCloud
``` Shell
# download latest nextcloud version
wget -O /tmp/nextcloud.zip https://download.nextcloud.com/server/releases/latest.zip
# extract downloaded nextcloud archive
sudo unzip -q /tmp/nextcloud.zip -d /var/www
# set the owner of the new nextcloud directory to www-data
sudo chown -R www-data:www-data /var/www/nextcloud
```
Restart Apache Service
``` Shell
# enable the nextcloud config
sudo a2ensite nextcloud
# enable required apache modules
sudo a2enmod rewrite headers env dir mime dav
# restart apache2 service for the changes to take effect
sudo service apache2 restart
```
Create account and Log-in NextCloud With The Created Credentials.
<p align="center">
  <img width="100%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/Screenshot%202024-09-16%20234114.png?raw=true">
  <br> Figure 5: NextCloud
</p>

### 5. Remote Access on LAN Level Under Same WiFi via Port Forwarding
**WSL’s Networking**
<p align="center">
  <img width="70%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/image2.png?raw=true">
  <br> Figure 6: WSL Networking
</p>

**LEGENDS**

- Device A | Server (192.168.1.85) - contains the WSL service
- Device B | Client (192.168.1.60) - tries to access Device A remotely
- Client sends → Server receives
1. Ensure pings from end-to-end devices work. If pings do not work, go to firewall settings to allow ICMP protocol policy.
<p align="center">
  <img width="70%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/imag4.png?raw=true">
  <br> Figure 7: Ping
</p>
3. If pings are good, network layer is established, proceed to find WSL’s service IP and do port forwarding at Local machine CMD:
<p align="center">
  <img width="70%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/image3.png?raw=true">
  <br> Figure 8: WSL Service IP
</p>

Perform routing to LAN using port forwarding:
``` Shell
# HTTP port:80
netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=80 connectaddress=172.19.131.7
```

After port forwarding is done, there is a route from the virtualized WSL’s service IP to the local machine, so that it's exposed to the outer LAN. However, you still cannot remote access it from other LAN devices. Firewall Policy needs to be set, allow port 80 and port 3389 in the policy.
<p align="center">
  <img width="70%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/imageFIRE.png?raw=true">
  <br> Figure 9: Firewall Policy
</p>

Remotely accessing NextCloud is actually the true goal of this project. Since HTTP port 80 is forwarded and firewall policy is set, can try to access the domain - http://192.168.1.85/nextcloud. However, you will get a page like such:
<p align="center">
  <img width="70%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/Screenshot%202024-10-21%20221558.png?raw=true">
  <br> Figure 10: Domain Restricted
</p>
Navigate to the config file of NextCloud and add the local IP domain to the trusted domain:
<p align="center">
  <img width="70%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/imageConfig.png?raw=true">
  <br> Figure 11: Config Chnages
</p>
Now NextCloud from Device A can be access successfully from Device B through LAN
<p align="center">
  <img width="70%" src="https://github.com/Yong-Wai-Chun/HomeLab-1-Cloud-based-Storage-Server/blob/main/img/462649868_1309292727146690_6593062607329030479_n.jpg?raw=true">
  <br> Figure 12: LAN Remote Access Success
</p>

### 6.1. Remote Access on WAN level with Different WiFi through Port Forwarding [FAILED!]
### 6.2. Remote Access on WAN level with Different WiFi through TailScale VPN Tunneling System [SUCCESS!]
