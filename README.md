# Overview
A simulated wireless CTF setup without the wireless signals.

CTF players SSH into a Kali docker container, which has a few wireless interfaces available to them. They can use these interfaces to scan and attack access points and clients which are running on the host system. All wireless packets are simulated using the mac80211_hwsim driver, enabling this setup to be used for events hosted in locations that do not allow wireless devices.

Each time the CTF is restarted, the player container is reset to the initial state removing any data created by the previous players.

Note: This project was used in a VM dedicated to the CTF and makes changes to host system settings where necessary. If you plan to use this on your everyday system, please review the contents of start.sh first!

# Challenge Configurations

## AP-Guest
This is a recon challenge consisting of just an open AP with some clients connected to it. The clients macs have been changed to mimic some mobile devices. The flag was the mac address of the Android device.

## AP-Bridged
A WPA2 AP with a client connected. This AP is bridged to eth1, which was connected to an internal network with other non-wireless CTF challenge systems. This flag is the WPA2 PSK. Once the PSK was cracked, the players can connect to this AP and access the other systems. This configuration currently assumes that there's a DHCP server running on the bridged network to provide the IP address.

## AP-WPA3
This is a WPA3 online bruteforce attack. No clients are connected. The wacker project from [Blunderbuss-WCTF](https://github.com/blunderbuss-wctf/wacker) was provided for the players to perform the bruteforce attack. The flag is the WPA3 passphrase.

## AP-Hidden
This is an AP with a hidden SSID on 5GHz, with a client connected. Players need to decloak the SSID and submit that as the flag.


### 

# Installation and Setup
These instructions cover the installation of dependencies for the host system and creating the docker image that provides the CTF player environment.

This project includes an example of an access point that is connected to another network outside of the host system using a bridged interface. If you intend to use this example, you will need a 2nd network interface.

This project has been successfully tested on Kali and Ubuntu versions:
- Kali 2020.3
- Kali 2021.1

- Ubuntu 20.04

This project was used in a VM dedicated to the CTF with 20GB storage, 2 cores, and 2GB RAM.

# Dependencies
The following are the dependencies for the host system.

```
sudo apt install docker.io
sudo usermod -aG docker $USER
sudo apt install net-tools
sudo apt install hostapd
sudo apt install bridge-utils
sudo apt install dnsmasq
sudo apt install wpasupplicant
sudo apt install macchanger
```
Note: You will need to logout and log back in after adding the user to the docker group.

# Docker Image
The CTF players access the challenges by connecting to the docker image via SSH. The dockerfile in build/ defines the tools to be installed and accessible to the players. The default login is root:ctf1234 and can be changed in the dockerfile.

Create the docker image:
```
docker build -t ctf-kali -f build/Dockerfile .
```

# Start the CTF
Update the start.sh script to use the correct interface names for the connection the players will be using (ENTRY_INTERFACE), and the interface for the bridged connection the the rest of the CTF network (CTF_INTERFACE).
You can also update the SSH port that the players will connect to in this script (PLAYER_SSH_PORT). By default it is set to 22.

You can start/restart the container, the APs, and clients by running the start.sh script.
Note: This project was used in a VM dedicated to the CTF and makes changes to host system setting where necessary. If you plan to use this on your everyday system, please review the contents of start.sh first!
```
sudo ./start.sh
```

# Start the CTF on boot
This project includes a systemd service file that can be used to start the CTF when the system boots. Edit start-ctf.service to update the following line to match the location of start.sh on your system:
```
ExecStart=/home/ctfadmin/wifi-ctf/start.sh
```

Copy the file to /etc/systemd/system:
```
sudo cp start-ctf.service /etc/systemd/system/
```

Enable the service:
```
sudo systemctl enable start-ctf
```

# Verify setup is working correctly
Connect to the container via SSH (if you changed the port in start.sh make sure you use that here):
```
ssh root@<ipaddress>
```
Default password is ctf1234

Put one of the wireless interfaces into monitor mode:
```
iwconfig wlan0 mode monitor
```

Scan for the APs and clients:
```
airodump-ng wlan0
```
You should see 3 APs here - AP-Guest, AP-Bridged, and AP-WPA3. It may take some time, but you should also see 6 clients associated with the Guest network, and 1 client for the AP-Bridged network.

Switch to 5GHz and scan for the remaining hidden network and it's client:
```
airodump-ng --band a wlan0
```

To verify that the bridge is working correctly, connect to AP-Bridged using wpa_supplicant

Example config:
```
ctrl_interface=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
	ssid="AP-Bridged"
	psk="thebridge"
	key_mgmt=WPA-PSK
	pairwise=CCMP TKIP
	group=CCMP TKIP
	proto=RSN
}
```
```
wpa_supplicant -c example.config -i wlan1
```

Once connected, request an IP address:
```
dhclient wlan1
```

The systems on this network should be reachable. You can test this with ping.


# TODO
- Make it easier to add/remove challenges
- Generation scripts for challenge types
- PSK randomization on boot
- Logging
