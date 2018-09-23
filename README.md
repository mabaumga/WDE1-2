# Einleitung

Python Skript zum Auslesen des Wetterdatenempfänger WDE1-2 von ELV, Versenden via MQTT und Empfangen per openHAB.

# Protokoll des Senders

Jede Zeile stellt einen kompletten Datensatz dar, der aus 25 durch Semikolon getrennten Feldern besteht. Die ersten drei Felder sind unveränderlich, dann folgen die Temperaturmesswerte (in °C) von acht Sensoren (z.B. S 300) und dann die Feuchtewerte (%) dieser acht Sensoren. Danach kommen Temperatur (°C), Luftfeuchte (%), Windgeschwindigkeit (km/h), Niederschlag (Wippenschläge) und Regensensor (0/1) des Kombisensors KS 200 oder KS 300. Das letzte Feld mit dem unveränderlichen Wert 0 steht für das Ende des Datensatzes.

Beispiel:

```
$1;1;;;;;11,3;19,9;;;;;;;99;54;;;;10,6;60;1,3;168;0;0
$1;1;;;;;11,3;19,9;;;;;;;99;54;;;;10,6;60;1,3;168;0;0
$1;1;;;;;11,3;19,9;;;;;;;99;54;;;;10,6;60;1,3;168;0;0
$1;1;;;;;11,3;19,9;;;;;;;99;54;;;;10,6;60;3,0;168;0;0
```

# Skript installieren

Der WDE Empfänger ist angeschlossen an den Raspberry wo auch Openhab installiert ist. Zusätzlich müssen auf diesem alle notwendigen Python Bibliotheken installiert werden:

```
pip install paho-mqtt
pip install PyMySQL
pip install PySerial
```

Das Skript unter src/main/python/readWDE12.python auf den Server (z.B. unter /home/openhabian/wde12/readWDE.py) kopieren. Bitte die brokerURL entsprechend dem eigenen Mosquitto Broker anpassen. Anschließend die Datei ausführbar machen:

```
sudo chmod 755 /home/openhabian/wde12/readWDE.py
```

# Skript als Service starten


Dazu eine neue Datei anlegen:
```
sudo nano /etc/init.d/wde12
```

Folgenden Inhalt in die neue Datei einfügen:
```
#! /bin/sh
# /etc/init.d/wde12

### BEGIN INIT INFO
# Provides:          /etc/init.d/wde12
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start daemon at boot time
# Description:       Enable service provided by daemon.
### END INIT INFO
 
case "$1" in
start)
echo "Starting WDE12"
# run application you want to start
python /home/openhabian/wde12/readWDE.py &
;;
stop)
echo "Stopping WDE12"
# kill application you want to stop
killall python
;;
*)
echo "Usage: /etc/init.d/wde12 {start|stop}"
exit 1
;;
esac
 
exit 0
```

Ausführbar machenund init.d bekannt machen:
```
sudo chmod 755 /etc/init.d/wde12
sudo update-rc.d wde12 defaults
```

Referenz für das Starten des Skriptes als Service: http://www.pietervanos.net/knowledge/start-python-script-from-init-d/

# Konfiguration von OpenHab2

Mit Openhab2 können wir uns wieder an dem Topic von Mosquitto lauschen und die Werte empfangen. Die Konfiguration liegt unter Openhab2-Config/services/mqtt.cfg und ist ziemlich einfach:

```
#
# Define your MQTT broker connections here for use in the MQTT Binding or MQTT
# Persistence bundles. Replace <broker> with an ID you choose.
#
  
mqtt:mosquitto.url=tcp://brokerURL:1883
  
# Optional. Client id (max 23 chars) to use when connecting to the broker.
# If not provided a default one is generated.
mqtt:mosquitto.clientId=openhab
```

Achtung: Hier die die BrokerURL anzupassen.
