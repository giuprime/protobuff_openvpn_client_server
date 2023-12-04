# protobuff_openvpn_client_server
Il codice generato ha lo scopo di chiarire i concetti base della comunicazione in oggetto, dando spunto alla possibile implementazione finale

### Descrizione
## Descrizione
Paho-mqtt è stato implementato principalmente nel file *./personal-mqtt.rb*, in modo da stampare messaggi chiari durante l'esecuione e, per chiarezza e controllo, i messaggi di tutti i client mqtt vengono inviati in sequenza  

Inoltre la classe presente nel file *mt32.rb*, ovvero il device mt32, come primo messaggio attende la configurazione (in cui troverà l'id della sessione). In tale file vengono creati 3 client che attendono la configurazione e, una volta che **tutti** l'hanno ricevuta, inviano un messaggio ogni secondo.

Infine il front-end, una volta avviato, ascolta sul topic della sessione con id ***sessionXXX*** per poi inviare la configurazione a due client con tale sessione, e ad un terzo con una sessione differente (ovvero il front-end non riceverà i dati di quest'ultimo). Questo solo a titolo di esempio dato che la configurazione di ogni device verrà inviata da front-end differenti (oppure dal server)

### Prerequisiti
## Prerequisiti

È necessario installare Ruby e [paho-mqtt](https://github.com/RubyDevInc/paho.mqtt.ruby)  

### Broker
## Broker

Per la scelta del broker è possibile utilizzare iot.eclipse.org oppure avviare un broker locale come [Mosquitto](https://mosquitto.org/) che è possibile installarlo tramite brew

@@ -31,11 +31,30 @@ Avviarlo caricando la configurazione
/usr/local/sbin/mosquitto -c /usr/local/etc/mosquitto/mosquitto.conf
```

#### Mosquitto example
### Configurazione

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
