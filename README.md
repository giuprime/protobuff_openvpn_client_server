# protobuff_openvpn_client_server
This example will show how to create two linux containers, the client and the server that communicate each other with the use of the overlay network of OpenVPN. We create the client the server and after we send some file for understanding which communication protocoll between JSON and Protobuffer is better.

### Description
## Description
First we create install protobuff, after we create the server and client for our scope, finally we send the messages between these two entities  

### Requirements
## Requirements
1. This tutorial is created for the distro Ubuntu 20.04
2. Installation of protobuffer protocol
3. OpenVPN
4. Easy-rsa

### Installation of Linux container
## Installation of Linux container

First we need to install the Linux container enviornment: [linux_container](https://linuxcontainers.org/it/lxd/getting-started-cli/) 
```
sudo apt-get install lxd lxd-client
```
Configuration of LXD:
```
sudo lxd init
```
Creations of the container:
```
lxc init ubuntu:20.04 container_name 
```
Starting the container:
```
lxc start container_name 
```
Use the container:
```
lxc exec container_name -- sudo --login --user root
```
### Installation of protobuff in the container
## Installation of protobuff in the container
Make sure you grab the latest version 
```
wget https://github.com/protocolbuffers/protobuf/releases/download/v24.4/protoc-24.4-linux-x86_64.zip
```
A little disclimer in macbook with chip apple silicon the vm ubuntu need an ISO image with arm architecture. This
created a problem in the installation of protobuf. So for users with a mac-book with chip apple silicon is needed to use the following command instead
```
wget https ://github .com/protocolbuffers/protobuf/releases/download/v25.1/protoc-25.1-linux-aarch_64.zip
```
Unzip
```
unzip protoc-24.4-linux-x86_64.zip -d protoc3
```
In the caso of MacBook:
```
unzip protoc -25.1 - linux - aarch_64 .zip -d protoc3
```
Move protoc to /usr/local/bin/
```
sudo mv protoc3/bin/* /usr/local/bin/
```
Move protoc3/include to /usr/local/include/
```
sudo mv protoc3/include/* /usr/local/include/
```
### Creation and setup of the server
## Creation and setup of the server
Install openVPN:
```
sudo apt update
sudo apt upgrade

sudo apt install openvpn
```
Install easy-rsa:
```
sudo apt install easy-rsa
```
Setup CA dir for OpenVpn: 
```
make-cadir ~/openvpn-ca
```
Configure CA variable: 
```
cd ~/openvpn-ca; nano vars;
```
Configure CA variable: 
```
cd ~/openvpn-ca; nano vars
```
Change variables in the file:
```
set_var KEY_COUNTRY=""
set_var KEY_PROVINCE=""
set_var KEY_CITY=""
set_var KEY_ORG=""
set_var KEY_EMAIL=""
set_var KEY_OU=""
set_var KEY_NAME=""
```
Update system’s variables: 
```
./easyrsa init-pki; ./easyrsa build-ca
```
Generate Diffie-Helmann keys: 
```
./easyrsa gen-dh
```
Generate server cretificate: 
```
./easyrsa build-server-full server nopass
```
Generate HMAC signature to increase the security: 
```
openvpn --genkey --secret pki/ta.key
```
Generate OpenVPN Revocation Certificate: 
```
./easyrsa gen-crl
```
Copy certificates in openvpn folder to configure server: 
```
cp -rp ./pki/{ca.crt,dh.pem,ta.key,crl.pem,issued,private} /etc/openvpn/server/
```
Create the configuration file of the server:
```
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf
sudo nano /etc/openvpn/server.conf
```
Change the configuration file server.conf:
```
comment server-bridge and also server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100 lines
tls-auth ta.key 0    --> uncomment
key-direction 0      --> add
cipher AES-256-CBC   --> uncomment
auth SHA512          --> add
user nobody          --> uncomment
group nogroup        --> uncomment
server 10.8.0.0 255.255.255.0  --> uncomment
client-to-client     --> add 
```
Enable network forwarding:
```
sudo nano /etc/sysctl.conf

sudo sysctl -p
```
Configure the NAT via file:
```
nano /etc/ufw/before.rules

You have to add at the begin of the file: 
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
```
Change another file:
```
sudo nano /etc/default/ufw -> Modify DEFAULT_FORWARD_POLICY="ACCEPT"
```
Allow forwarded traffic to go through firewall:
```
sudo ufw allow 1194/udp
sudo ufw allow OpenSSH
sudo ufw disable
sudo ufw enable
```
Copy the certification and the other file in the folder of openVPN:
```
mv ./openvpn-ca/pki/ca.crt /etc/openvpn
6 mv ./openvpn-ca/pki/issued/server.crt /etc/openvpn
7 mv ./openvpn -ca/pki/private/server.key /etc/openvpn
8 mv ta.key /etc/openvpn/
```
Enable OpenVPN Service
```
sudo systemctl enable openvpn@server
``` 
Start  OpenVPN Service:
```
sudo systemctl start openvpn@server
```
If there is a warning with the IPv4 and IPv6 you must change the server.conf file in this way:
```
# TCP or UDP server ?
#;proto tcp
#;proto tcp4
proto udp4
```
Now the server is created, but we need to create the client with the command of LXC.
After the creation of the client we need to send the certification from the server to the client in this way:
```
scp -r /etc/openvpn/files root@CLIENTIP :/etc/openvpn
```
Send this following file:
1. dh.pem
2. ta.key
3. ca.crt
4. server.crt
5. server.key
### Creation and setup of the client
## Creation and setup of the client
Install Openvpn: 
```
sudo apt install openvpn
```
Copy template config file: 
```
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf client-configs/base.conf
```
Create config file:
```
nano ./client-configs/base.conf 

remote [server_ip] 1194 --> add
user nobody             --> uncomment  
group nogroup           --> uncomment 
cipher AES-256-CBC      --> add
auth SHA512             --> add
key-direction 1         --> add 
script-security 2       --> add
up /etc/openvpn/update-resolv-conf   --> add
down /etc/openvpn/update-resolv-conf --> add
```
Copy in OpenVPN root:
```
mv base.conf /etc/openvpn/user1.ovpn
```
Copy in OpenVPN root: 
```
mv user1.ovpn /etc/openvpn/
```
Launch Openvpn client: 
```
sudo openvpn --config user1.ovpn
```
Now client and server communicate using the same overlay network.

