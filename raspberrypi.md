---
ref: raspberrypi
lang: de
navorder: 3
navtitle: Raspberry Pi
---
## Raspberry Pi Variante
[<img src="img/smd-3drpi.png" width="200" alt="RPI" title="RPI">](img/smd-3drpi.jpg)  
In der Raspberry Pi Variante wird der Adapter über die GPIO Buchse direkt auf den [Raspberry Pi](https://www.raspberrypi.org/) aufgesteckt.

Achtung! Abhängig von der Adapter Version kann die Buchse auch kürzer ausfallen, jedoch ist die Platine immer bündig am "unteren Ende" (Pin 1)
aufzusetzen.

Dabei erfolgt die Stromversorgung des Adapters direkt vom Raspberry Pi und die Kommunikation geht über die folgenden
Pins am [Raspberry Pi GPIO](https://www.raspberrypi.org/documentation/usage/gpio/):
* TxD: Pin 8 (GPIO 14)
* RxD: Pin 10 (GPIO 15)

Diese Pins sind dem physikalischen Device "serial0" zugeordnet.
Das physikalische device ist meist dem logischen Device ttyAMA0 zugeordnet, jedoch gibt es abhängig von der Rasperry Pi Version und der
Raspbian / Rasperry Pi OS Version immer wieder andere Zuordnungen. Die aktuelle Konfiguration kann so bestimmt werden:
* `ls -l /dev` ausführen und dann die Zuordnung von serial0 suchen.

Dies ist im Normalfall entweder ttyAMA0 oder ttyS0. Da der UART von ttyS0 Schwierigkeiten mit höheren Baudraten hat, wird dringen angeraten, <span style="color:OrangeRed">**unbedingt ttyAMA0 zu benutzen**</span>, wo diese Probleme nicht auftreten können.

Sollte serial0 mit ttyS0 verbunden sein, was vor allem beim Raspberry Pi 3 und manchen Raspbian/Rasperry Pi OS Versionen auftritt, dann können ttyS0 und ttyAMA0 durch einen Eintrag in /boot/config.txt vertauscht werden:
* `dtoverlay=pi3-miniuart-bt`

Die ebusd device Konfiguration in /etc/default/ebusd lautet daher:

* `-d ens:/dev/ttyAMA0 --latency=50`

Falls keine anderen Gründe dagegen sprechen, wird dringend empfohlen, das Enhanced Protokoll ("ens:") zu verwenden, da nur dieses die Einhaltung des korrekten Timings am eBUS garantiert. Trotzdem soll "latency=50" spezifiziert werden, um bei den oft ungünstig implementierten UART Treibern im Raspberry Pi OS den ebusd nicht in ein Timeout laufen zu lassen.

### Konfiguration mit Raspberry Pi OS
Standardmäßig werden die für den eBUS Adapter benötigten Pins vom Raspberry Pi OS für eine serielle Login Shell verwendet.
Um nun stattdessen den eBUS Adapter über diese Schnittstelle laufen lassen zu können, muss diese Shell stillgelegt werden.

Achtung! Wenn diese Shell nicht komplett stillgelegt wird, dann sendet sie unkontrollierte Daten auf den eBUS was zu einer Störung des gesamten Bus Systems führen kann.

Zur Stilllegung müssen folgende Schritte durchgeführt werden (sinngemäß, die Texte können je nach Raspberry Pi OS leicht variieren):
* `sudo raspi-config` ausführen
* `Interface Options` auswählen
* `Serial Port` auswählen
* `Would you like a login shell to be accessible over serial?` mit "Nein"/"No" beantworten
* `Would you like the serial port hardware to be enabled?` mit "Ja"/"Yes" beantworten
* "raspi-config" beenden und neu booten

#### Spezielle UART Einstellungen
In manchen Raspberry Pi Versionen und/oder Raspbian/Raspberry Pi OS Versionen ist kein UART per default aktiv.
Deshalb sollte kontrolliert werden, ob in `/boot/config.txt` der Eintrag

`enable_uart=1`

enthalten ist. Gegebenenfalls nachtragen und dann neu booten.

Achtung! Sollte der `ttyebus` Treiber vorhanden sein, so ist er komplett zu deinstallieren. Dieser Treiber ist für den Enhanced Mode nicht geeeignet.

Um zu verifizieren, dass die Einstellungen richtig sind, kann man `dmesg|grep ttyAMA0` ausführen. Dies sollte eine Zeile wie
folgt ausgeben:
```
[    1.166081] fe201000.serial: ttyAMA0 at MMIO 0xfe201000 (irq = 29, base_baud = 0) is a PL011 rev2
```
