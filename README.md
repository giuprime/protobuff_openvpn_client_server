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

First we need to install the Linux container enviornment: [linux_container]([(https://linuxcontainers.org/it/lxd/getting-started-cli/]) 
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
wget https :// github .com/ protocolbuffers / protobuf / releases / download /v25 .1/protoc-25.1-linux-aarch_64.zip
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
### Creation of the server
## Creation of the server
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
`
comment server-bridge and also server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100 lines
uncomment (removing ;) the blue line and add the red ones:
tls-auth ta.key 0
key-direction 0
cipher AES-256-CBC
auth SHA512
user nobody
group nogroup
server 10.8.0.0 255.255.255.0
client-to-client
`
