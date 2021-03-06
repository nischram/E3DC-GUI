## RSCP GUI
#### Wichtig
Bitte meine neue Repository __E3dcGui__ beachten!  
[https://github.com/nischram/E3dcGui.git](https://github.com/nischram/E3dcGui.git)  
Diese Repository wird nicht weitergeführt!

_Version 2.1 Stand: 22.07.2016_

Changelog V2.1: Watchdog erstellt

Hier beschreibe ich wie du auf einem Rasperry Pi mit 3,2" Touchdisplay dein _S10 Hauskraftwerk_ von _E3DC_ visualisieren kannst.

Als Schnittstelle zwischen S10 und dem Raspberry dient eine Beispielapplikation, die E3DC im Downloadbereich hat.

Ich nutze diese Applikation auf einem Raspberry Pi 3, mit eine Joy-It 3,2" Touchdisplay ([Conrad 1380381 - 62](https://www.conrad.de/de/raspberry-pi-display-modul-schwarz-rb-tft32-v2-raspberry-pi-a-b-b-raspberry-pi-1380381.html)). Mein S10-E12 hat Release-Version _2016-02_. Diese Applikation funktioniert auch für S10-Mini und für S10-SP40. Mit meiner sehr geringen Erfahrung, in der Programmierung, sind die Änderungen in den Applikationen eventuell unprofessionell ausgefallen, aber die Funktionalität wird erfüllt.


Bild noch mit Version 1 der RSCP GUI:
<img src="https://s20.postimg.org/90o1nmhy5/E3_DC_GUI.jpg" alt="E3DC-GUI">

### Info

Die Funktion der E3DC-GUI besteht aus zwei Programmen.

1. __RscpMain__  Die RSCP-Beispielapplication geschrieben in C++ von E3/DC
2. __RscpGui__   Ein einfaches Display Programm geschrieben in C

Das RSCP Programm holt per E3DC-Protokoll die Daten vom S10 und schreibt die Werte in eine Text-Datei in das RAMDisk des Raspberry Pi.

Die GUI liest die Text-Datei aus dem RAMDisk und baut die GUI.

Beide Programme liegen nach dem folgenden Download in diesem Ordner:
```
/home/pi/RscpGui/
```

### Netzwerk

Es ist Wichtig darauf zu achten, dass der Raspberry Pi im selben Netzwerk wie das S10 installiert ist. Der IP-Adressbereich muss gleich sein, zum Beispiel so: 192.168.178.xxx

Auch darf die Verbindung zwischen S10 und Raspberry Pi nicht über eine VPN Verbindung hergestellt werden.

### Download

Mit dem Git-Clone-Befehl wird der SourceCode auf den Raspberry kopiert.
```shell
git clone git://github.com/nischram/E3DC-GUI.git ~/RscpGui
```

### ToDo RscpMain.cpp

In der _RscpMain.c_ musst du die Zugangsdaten zum S10 noch definieren. Die Einstellungen sind in Zeile 22-25 vorzunehmen:
```
#define SERVER_IP           "192.168.xxx.xxx"
#define E3DC_USER           "Max.Mustermann"
#define E3DC_PASS           "password"
#define AES_PASS            "1234567890"
```
Die IP-Adresse ist die IP von deinem S10, E3DC_USER ist der Benutzername vom Kundenportal, E3DC_PASS ist dein Password zum Kundenportal und AES_PASS ist ein RSCP-Passwort welches du am S10 vergeben kannst. Im „Hauptmenü“ unter „Einstellungen“ kann das RSCP-Passwort gesetzt werden.

Da euer Hauskraftwerk mit Leistungsmesser für externe Quellen und / oder mit Wallbox ausgestattet sein kann, muss du noch definieren ob die Komponenten vorhanden sind. Dies geschieht in den Zeilen 28 bis 31:
```
//Wenn vorhandene, Bitte für Externe-Quelle (Additional) und
//Wallbox je ein 1, sonst 0 eintragen.
#define Additional  	1
#define Wallbox 		1
```

### Starten der RscpMain.cpp

Das wechseln in den Ordner nicht vergessen:
```
cd RscpGui
```
Nach dem Einstellen muss das Programm mit:
```
make
```
compiliert werden.

Ein Test der RscpMain führt man mit diesem Befehl aus:
```
./RscpMain
```
Die Ausgabe sollte wie folgt aussehen:
```
pi@raspberrypi:~/RscpGui $ ./RscpMain
Connecting to server 192.168.xxx.xxx:5033
Connected successfully

Request authentication
RSCP authentitication level 10

____________________
Request cyclic data
Serial-Number is S10-4013xxxxxxxx
System Time is 30.06.2016_20:35:47
System Unix-Time is 1467311062
PV Power is 46 W
Battery Power is -930 W
House Power is 974 W
Grid Power is -2 W
Autarky is 99 %
Self Consumption is 99 %
Additional Power is 0 W
Wallbox Power All is 0 W
Wallbox Power Solar is 0 W
Battery SOC is 20 %
Battery State = 1
PVI State = 1
LM0 State = 1
```

Diese Werte werden in der RscpGui.txt im RAMDisk gespeichert (siehe Abschnitt RAMDisk). Die Datei ist in Folgender Reihenfolge gpeichert: (Werte sind ausgedacht, ich habe keine Wallbox)

```shell
30.06.2016               //Datum
20:35:47                 //Zeitr
2260                     //Power PV
394                      //Power Battery (+=Laden -=Endladen)
478                      //Hausverbrauch
-8                       //Power Netz (+=Bezug -=Einspeisung)
26                       //Battery SOC (Füllstand)
1                        //Batterytatus >1 =Aktiv
92                       //Autarkie
96                       //Eigenverbrauch (Self Consuption)
S10-4013xxxxxxxx         //Seriennummer
1467311062               //Unix-Time
1                        //externe Quelle vorhanden (Additional) 1=Aktiv
1840                     //Power externe Quelle
1                        //Wallbox vorhanden 1=Aktiv
3220                     //Power Wallbox All
3200                     //Power Wallbox Solar
1                        //Status Wechselrichter (PVI) 1=Aktiv
1                        //Status Leistungsmesser 1=Aktiv
```

Gestoppt wird die Applikation vorerst mit „strg“ + „c“

### ToDo RscpGui.c

Die GUI-Datei braucht nur noch kompiliert werden, Änderungen sind nicht nötig.
```
gcc -g -o RscpGui  RscpGui.c
```

### Testen RscpGui

Die GUI kann mit folgendem Befehl getestet werden, aber die Werte werden ohne die RscpMain nicht aktuallisiert.
```
./RscpGui
```

Gestoppt wird die Applikation vorerst mit „strg“ + „c“

### Autostart der Applikation (durch Crontab-Job)
Damit das Programm und die Variablen nun dauerhaft genutzt werden können, muss die Applikation auf dem Raspberry in den Autostart gebracht werden. Dies geschieht bei einem Raspberry Pi in dem ein Crontab-Job erstellt wird.

Die Crontab ruft man auf mit:
```shell
pi@raspberrypi ~/RscpGui $  crontab -e
```
- Crontab für die Bearbeitung öffnen

In der geöffneten Crontab wird eine neue Zeile mit diesem Inhalt am Ende angefügt:
```shell
@reboot /home/pi/RscpGui/atReboot.sh
```
Die Änderung wird mit „ctrl“ + „o“ gespeichert und die Crontab beendet mit
„ctrl“ + „x“.
Weitere Informationen zur Crontab entnehmen Sie bitte aus Quellen wie z. B. dieser:
[https://raspberry.tips/raspberrypi-einsteiger/cronjob-auf-dem-raspberry-pi-einrichten](https://raspberry.tips/raspberrypi-einsteiger/cronjob-auf-dem-raspberry-pi-einrichten)

In der atReboot.sh wird mit einer Verzögerung die RscpMain und danach die RscpGui gestartet.
Die atReboot.sh muss noch ausführbar gemacht werden, mit:
```
chmod +x atReboot.sh
```

### Beenden

Damit du den Raspberry auch mit gestarteter Applikation ausschalten kannst, habe ich ein Programm erstellt, welches die 3.Taste meines Displays abfrgt. Nach einem druck von 2 Sekunden, fährt der Raspberry runter.
Die Datei hierfür heißt Key3.c auch diese muss Compiliert werden:
```
gcc -g -o Key3  Key3.c -l wiringPi
```

### RAMDisk

RAMDisk am Raspberry erstellen:
Zuerst wird ein Mountpoint für die RAM-Disk erzeugt:
```
sudo mkdir /mnt/RAMDisk
```
Für die RAM-Disk muß die Filesystem Table angepasst werden:
```
sudo nano /etc/fstab
```
Einfügen dieser Zeile am Ende der Datei:
```
tmpfs /mnt/RAMDisk tmpfs nodev,nosuid,size=4M 0 0
```
Die Größe wird über den Parameter "4M" auf 4 MB festgelegt. Jetzt montiert man alle Filesysteme über:
```
sudo mount -a
```
Der Erfolg lässt sich mit Diskfree überprüfen:
```
df
```
Es sollte dann ein Eintrag mit der RAM-Disk zu finden sein:
```
Filesystem 1K-blocks Used Available Use% Mounted on
rootfs 15071704 2734624 11674436 19% /
/dev/root 15071704 2734624 11674436 19% /
devtmpfs 218620 0 218620 0% /dev
tmpfs 44580 236 44344 1% /run
tmpfs 5120 0 5120 0% /run/lock
tmpfs 89140 0 89140 0% /run/shm
/dev/mmcblk0p1 57288 19712 37576 35% /boot
tmpfs 4096 0 4096 0% /mnt/RAMDisk
```

Diesen Teil zum RAMDisk habe ich von hier Kopiert:
[http://www.kriwanek.de/raspberry-pi/486-ram-disk-auf-raspberry-pi-einrichten.html](http://www.kriwanek.de/raspberry-pi/486-ram-disk-auf-raspberry-pi-einrichten.html)

### Watchdog

Teilweise bleibt die RSCP-Applikation hängen und die Automatische re-connection in der Applikation funktioniert leider nicht immer. So wird ein Neustart der RSCP-Applikation nötig.

Für dieses Problem habe ich einen einfachen WatchDog geschrieben. Damit der WatchDog den Betrieb der Applikation überwachen kann, lasse ich mit einem kleinen Teil in der RscpMain, eine weitere Datei im RAMDisk erstellen. In der Datei ist die Unixtime des S10, diese widerum liest der WatchDog ein und vergleicht diese mit einer definierten Differenz mit der aktuellen Ziet.  

In der RscpMain.cpp sind die Zeilen 93 bis 98 neu:
```
ofstream fout("/mnt/RAMDisk/Unixtime.txt");
if (fout.is_open()) {
  fout << TAG_EMS_OUT_DATE << ";" << TAG_EMS_OUT_TIME << "\n" << TAG_EMS_OUT_UNIXTIME << endl;
  fout.close();
  }
else cerr << "Konnte Unixtime.txt nicht erstellen!";
```
In der neuen Watchdog.cpp kann noch verschiedenes definiert werden:
```
//Zeitdifferenz zur aktuellen Zeit bis zur Watchdog aktivierung, in Sekunden
#define diff            300
//Zeitinterval für die Abfragen des Watchdog, in Sekunden
#define sleepTime       120
//Anzahl Programm Neustarts bis zum Reboot
#define rebootCounter   4
//Nach dieser Zeit wird der rebootCounter zurückgesetzt wenn die Daten aktuell sind, in Minuten
#define resetMin        60
```
Wenn der Watchdog zuschlägt, erstellt er eine Datei "Watchdog.csv" im RscpGui Ordner. Somit ist eine kotrolle der aktivität möglich. Es wird je aktivität eine Zeile erstellt, entweder mit reboot eintrag oder mit pkill wenn die Applikation neu gestartet wurde.

Für den automatischen Start habe ich den Watchdog in der _atReboot.sh_ eingefügt.

Damit einfach kompiliert werden kann habe ich das Makefile auch angepasst.

Somit müsste für ein nachrüsten des WatchDog 1. die _RscpMain.cpp_ angepasst, 2. die _Watchdog.cpp_ kopiert und 3. das _Makefile_ und die _atReboot.sh_ ausgetauscht werden.

ToDo: Ein Fehler ist aktuell noch zu beheben -- mehrfacher Reboot des PI --. Also solltet ihr das S10 vom Netzwek trennen oder eine längere sonstige Störung bestehen, würde alle 8 Minuten der Raspberry Pi mit einem Reboot neu gestartet. Um dies zu stoppen musst du folgendes in der Komandozeile eingeben:
```
pi@raspberrypi:~ $ pkill watchdog
```

### Dateibeschreibung

AES.cpp / AES.h / RscpProtokoll.cpp / RscpTag.h / RscpType.h / SocketConnektion.cpp / SocketConnection.h
gehören zur RSCP-Applikation.

font_8x8.c / framebuffer.c / touch.c / touch.h
gehören zur GUI und ich habe den Ursprung dieser Datein hier her:
[http://ozzmaker.com/controlling-the-gpio-on-a-raspberry-pi-with-a-touchscreen/](http://ozzmaker.com/controlling-the-gpio-on-a-raspberry-pi-with-a-touchscreen/)

### Änderungen

Änderungen an meiner GUI dürfen gerne Vorgenommen und hier eingefügt werden.

### Nutzen der RSCP-Applikation

Die RSCP-Applikation (RscpMain.cpp) kann für viele weitere Anwendungen genutzt werden. Wie zum Beispiel meine [E3DCtoHM](https://github.com/nischram/E3DCtoHM.git) um eine HomeMatic mit Werten des S10 zu versorgen.

Wenn ihr eigene Projekte erstellt, würde ich mich freuen, wenn ihr diese veröffentlicht.

### Meine Quelle:

Downloadbereich E3DC Kundenportal [https://s10.e3dc.com](https://s10.e3dc.com)

Service > Download > Zusätzliche Optionen >  RSCP-Beispielapplikation_2016-04-25.pdf

[Link](https://s10.e3dc.com/s10/module/download/get.php?dl=3408) (Benutzerdaten erforderlich)
