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

# Unzip
unzip protoc-24.4-linux-x86_64.zip -d protoc3

# Move protoc to /usr/local/bin/
sudo mv protoc3/bin/* /usr/local/bin/

# Move protoc3/include to /usr/local/include/
sudo mv protoc3/include/* /usr/local/include/

# Optional: change owner
sudo chown $USER /usr/local/bin/protoc
sudo chown -R $USER /usr/local/include/google




### Mosquitto example
## Configurazione

Subscription with `mosquitto_sub -t "/World"` and publish with `mosquitto_sub -t "/World" -m "World"`
Broker mqtt ip `iot.eclipse.org` e porta `1883`.

### Esecuzione
Per il supporto di Mosquitto sia alle richieste mqtt che quelle su websocket, è necessario modificare la configurazione come segue
```
#port 1883
listener 1883
listener 9001
...
#max_connections -1
...
#protocol mqtt
protocol websockets
```
In questo modo la porta 1883 sarà adibita per le connessioni mqtt su TCP, mentre la 9001 per mqtt su websocket

### Mosquitto esempi

È possibile eseguire mosquitto da terminale effettuando la sottoscrizione con `mosquitto_sub -t "/World"` e publicando con `mosquitto_sub -t "/World" -m "Hello"`

## Esecuzione

Per l'esecuzione devono essere eseguiti in ordine il server, i dispositivi MT32 (che attendono la configurazione) e infine il front-end (che invia la configurazione)

@@ -49,6 +68,6 @@ ruby mt32.rb
ruby front-end.rb
```

### TODO
## TODO

Non è stata implementato l'invio della configurazione qualora il device venga acceso dopo la creazione della sessione (basta che il front-end si sottoscriva al topic *azienda/+* e, alla ricezione di un messaggio, invia la configurazione se presente)
  13 changes: 11 additions & 2 deletions13  
personal-mqtt.rb
@@ -35,8 +35,17 @@ def initialize
    end

    # CONNECTING TO THE BROKER
    # self.connect('localhost', 1883)
    self.connect('iot.eclipse.org', 1883)
    # ip = 'iot.eclipse.org'
    # port = 1883
    ###
    ip = 'localhost'
    port = 1883
    ###
    # ip = 'broker.mqttdashboard.com'
    # port = 8000

    puts "Connecting #{ip} @ #{port}"
    self.connect(ip, port)
  end

  def subscribe(*topics)
