# Python-Anwendungen unter Ubuntu und Raspberry Pi OS automatisch starten

> **Ziel:** Eine beliebige Python-Anwendung unter Linux als `systemd`-Service einrichten, sodass sie automatisch beim Systemstart oder beim Benutzer-Login startet und bequem über die Konsole gestartet, gestoppt, neu gestartet und überwacht werden kann.

Diese Anleitung gilt insbesondere für:

- Ubuntu
- Ubuntu Server
- Raspberry Pi OS (früher „Raspbian“)
- Raspberry Pi OS Lite
- andere Debian-basierte Linux-Systeme mit `systemd`

Die Beispiele verwenden eine Python-Anwendung namens `my-python-app`. Pfade, Benutzername und Startdatei müssen an das eigene Projekt angepasst werden.

---

## Inhaltsverzeichnis

1. [Warum systemd?](#1-warum-systemd)
2. [System-Service oder User-Service?](#2-system-service-oder-user-service)
3. [Beispiel-Projektstruktur](#3-beispiel-projektstruktur)
4. [Python-Umgebung vorbereiten](#4-python-umgebung-vorbereiten)
5. [Anwendung manuell testen](#5-anwendung-manuell-testen)
6. [System-Service einrichten](#6-system-service-einrichten)
7. [Service verwalten](#7-service-verwalten)
8. [Logs anzeigen](#8-logs-anzeigen)
9. [User-Service als Alternative](#9-user-service-als-alternative)
10. [User-Service bereits beim Booten starten](#10-user-service-bereits-beim-booten-starten)
11. [Besonderheiten unter Raspberry Pi OS](#11-besonderheiten-unter-raspberry-pi-os)
12. [Umgebungsvariablen verwenden](#12-umgebungsvariablen-verwenden)
13. [Anwendung nach einem Update neu starten](#13-anwendung-nach-einem-update-neu-starten)
14. [Service wieder entfernen](#14-service-wieder-entfernen)
15. [Fehlersuche](#15-fehlersuche)
16. [Empfohlene Konfiguration](#16-empfohlene-konfiguration)
17. [Befehlsübersicht](#17-befehlsübersicht)

---

# 1. Warum systemd?

`systemd` ist auf Ubuntu und Raspberry Pi OS der Standard-Service-Manager.

Damit kann eine Python-Anwendung:

- automatisch beim Booten gestartet werden,
- unabhängig von einer geöffneten Konsole laufen,
- bei einem Fehler automatisch neu gestartet werden,
- sauber gestoppt und neu gestartet werden,
- Logs über `journalctl` bereitstellen,
- unter einem normalen Benutzer statt als `root` laufen.

Nach der Einrichtung reichen beispielsweise folgende Befehle:

```bash
sudo systemctl status my-python-app
sudo systemctl restart my-python-app
sudo systemctl stop my-python-app
sudo journalctl -u my-python-app -f
```

---

# 2. System-Service oder User-Service?

Es gibt zwei sinnvolle Varianten.

## Variante A: System-Service

Service-Datei:

```text
/etc/systemd/system/my-python-app.service
```

Steuerung:

```bash
sudo systemctl start my-python-app
sudo systemctl stop my-python-app
sudo systemctl restart my-python-app
```

### Geeignet für

- Server-Anwendungen
- Backends
- APIs
- Netzwerkdienste
- Programme auf Raspberry Pis
- Anwendungen, die bereits beim Booten laufen sollen
- Produktivsysteme

### Empfehlung

Für dauerhaft laufende Python-Dienste ist ein **System-Service normalerweise die beste Wahl**.

---

## Variante B: User-Service

Service-Datei:

```text
~/.config/systemd/user/my-python-app.service
```

Steuerung:

```bash
systemctl --user start my-python-app
systemctl --user restart my-python-app
```

### Geeignet für

- Entwicklungsrechner
- Desktop-Anwendungen
- Dienste, die einem bestimmten Benutzer gehören
- Anwendungen, die normalerweise erst mit einem Benutzer gestartet werden sollen

Ein User-Service kann mit `linger` ebenfalls bereits beim Booten gestartet werden. Dazu später mehr.

---

# 3. Beispiel-Projektstruktur

Angenommen, die Anwendung befindet sich unter:

```text
/home/myuser/apps/my-python-app/
```

Beispiel:

```text
my-python-app/
├── .venv/
├── requirements.txt
├── config/
├── logs/
└── src/
    └── main.py
```

Der Benutzer heißt im Beispiel:

```text
myuser
```

Die Anwendung soll mit folgender Datei gestartet werden:

```text
src/main.py
```

---

# 4. Python-Umgebung vorbereiten

## 4.1 Ubuntu

Zuerst die Paketlisten aktualisieren:

```bash
sudo apt update
```

Python und die benötigten Werkzeuge installieren:

```bash
sudo apt install python3 python3-venv python3-pip
```

Projektordner öffnen:

```bash
cd /home/myuser/apps/my-python-app
```

Virtuelle Umgebung erstellen:

```bash
python3 -m venv .venv
```

Aktivieren:

```bash
source .venv/bin/activate
```

Optional `pip` aktualisieren:

```bash
python -m pip install --upgrade pip
```

Abhängigkeiten installieren:

```bash
pip install -r requirements.txt
```

Virtuelle Umgebung verlassen:

```bash
deactivate
```

---

## 4.2 Raspberry Pi OS

Raspberry Pi OS enthält Python 3 bereits standardmäßig.

Für zusätzliche Python-Pakete sollte auf aktuellen Raspberry-Pi-OS-Versionen eine virtuelle Umgebung verwendet werden.

Benötigte Pakete installieren:

```bash
sudo apt update
sudo apt install python3-venv python3-pip
```

Falls beim Erstellen einer virtuellen Umgebung Python-Komponenten fehlen, kann zusätzlich installiert werden:

```bash
sudo apt install python3-full
```

Projekt öffnen:

```bash
cd /home/myuser/apps/my-python-app
```

Virtuelle Umgebung erstellen:

```bash
python3 -m venv .venv
```

Aktivieren:

```bash
source .venv/bin/activate
```

Abhängigkeiten installieren:

```bash
pip install -r requirements.txt
```

Danach:

```bash
deactivate
```

> **Hinweis:** Pakete sollten auf aktuellen Raspberry-Pi-OS-Versionen nicht mit einem normalen `sudo pip install ...` direkt in das System-Python installiert werden. Eine `venv` vermeidet Konflikte mit den durch Debian verwalteten Python-Paketen.

---

# 5. Anwendung manuell testen

Bevor ein Service erstellt wird, sollte exakt der spätere Startbefehl manuell getestet werden.

Beispiel:

```bash
cd /home/myuser/apps/my-python-app
```

Dann:

```bash
/home/myuser/apps/my-python-app/.venv/bin/python /home/myuser/apps/my-python-app/src/main.py
```

Falls die Anwendung als Python-Modul gestartet wird:

```bash
/home/myuser/apps/my-python-app/.venv/bin/python -m package_name
```

oder beispielsweise:

```bash
/home/myuser/apps/my-python-app/.venv/bin/python -m package_name.main
```

Der verwendete Befehl sollte vollständig funktionieren, bevor `systemd` eingerichtet wird.

> **Wichtig:** In einer `systemd`-Service-Datei muss eine virtuelle Umgebung nicht mit `source .venv/bin/activate` aktiviert werden. Stattdessen wird direkt der Python-Interpreter aus der virtuellen Umgebung verwendet.

---

# 6. System-Service einrichten

## 6.1 Service-Datei erstellen

```bash
sudo nano /etc/systemd/system/my-python-app.service
```

Beispielinhalt:

```ini
[Unit]
Description=My Python Application
After=network-online.target
Wants=network-online.target

[Service]
Type=simple

User=myuser
Group=myuser

WorkingDirectory=/home/myuser/apps/my-python-app

ExecStart=/home/myuser/apps/my-python-app/.venv/bin/python /home/myuser/apps/my-python-app/src/main.py

Restart=on-failure
RestartSec=5

Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

Anschließend speichern:

```text
Ctrl+O
Enter
Ctrl+X
```

---

## 6.2 Bedeutung der wichtigsten Optionen

### `Description`

```ini
Description=My Python Application
```

Eine frei wählbare Beschreibung des Services.

---

### `After=network-online.target`

```ini
After=network-online.target
Wants=network-online.target
```

Der Service wird erst gestartet, nachdem `systemd` das Netzwerk als online betrachtet.

Das ist besonders sinnvoll für:

- Backends
- TCP-/UDP-Server
- Webserver
- MQTT-Anwendungen
- Datenbank-Clients
- Netzwerkkommunikation

Für Anwendungen ohne Netzwerkabhängigkeit können diese beiden Zeilen weggelassen werden.

---

### `User`

```ini
User=myuser
```

Die Anwendung läuft unter diesem Benutzer.

Eine Python-Anwendung sollte normalerweise **nicht als `root` ausgeführt werden**, sofern dies nicht zwingend erforderlich ist.

---

### `Group`

```ini
Group=myuser
```

Legt die primäre Gruppe des Prozesses fest.

---

### `WorkingDirectory`

```ini
WorkingDirectory=/home/myuser/apps/my-python-app
```

Legt das Arbeitsverzeichnis der Anwendung fest.

Das ist besonders wichtig, wenn die Anwendung relative Pfade verwendet.

Beispiel:

```python
open("config/settings.json")
```

Dieser Pfad wird relativ zum `WorkingDirectory` ausgewertet.

---

### `ExecStart`

```ini
ExecStart=/home/myuser/apps/my-python-app/.venv/bin/python /home/myuser/apps/my-python-app/src/main.py
```

Das ist der eigentliche Startbefehl.

Hier sollten **absolute Pfade** verwendet werden.

---

### `Restart`

```ini
Restart=on-failure
```

Die Anwendung wird neu gestartet, wenn sie unerwartet mit einem Fehler endet.

Das ist für die meisten Serveranwendungen eine gute Einstellung.

Alternativ:

```ini
Restart=always
```

Damit wird die Anwendung auch nach einem normalen Programmende erneut gestartet.

---

### `RestartSec`

```ini
RestartSec=5
```

Wartet fünf Sekunden vor einem automatischen Neustart.

Das verhindert schnelle Endlosschleifen, falls die Anwendung direkt nach dem Start wieder abstürzt.

---

### `PYTHONUNBUFFERED`

```ini
Environment=PYTHONUNBUFFERED=1
```

Python schreibt Ausgaben dadurch ohne zusätzliche Pufferung.

`print()`-Ausgaben erscheinen daher schneller im `journalctl`-Log.

---

## 6.3 systemd-Konfiguration neu laden

Nach dem Erstellen oder Ändern einer Service-Datei:

```bash
sudo systemctl daemon-reload
```

---

## 6.4 Service beim Booten aktivieren

```bash
sudo systemctl enable my-python-app
```

Damit wird die Anwendung bei zukünftigen Systemstarts automatisch gestartet.

---

## 6.5 Service sofort starten

```bash
sudo systemctl start my-python-app
```

Alternativ können Aktivieren und Starten kombiniert werden:

```bash
sudo systemctl enable --now my-python-app
```

---

# 7. Service verwalten

## Status anzeigen

```bash
sudo systemctl status my-python-app
```

---

## Starten

```bash
sudo systemctl start my-python-app
```

---

## Stoppen

```bash
sudo systemctl stop my-python-app
```

---

## Neu starten

```bash
sudo systemctl restart my-python-app
```

Das ist beispielsweise nach einer Änderung am Python-Code sinnvoll.

---

## Konfiguration neu einlesen

Wenn nur Python-Dateien geändert wurden, ist kein `daemon-reload` notwendig:

```bash
sudo systemctl restart my-python-app
```

Wenn dagegen die `.service`-Datei verändert wurde:

```bash
sudo systemctl daemon-reload
sudo systemctl restart my-python-app
```

---

## Autostart deaktivieren

```bash
sudo systemctl disable my-python-app
```

---

## Stoppen und gleichzeitig Autostart deaktivieren

```bash
sudo systemctl disable --now my-python-app
```

---

## Wieder aktivieren

```bash
sudo systemctl enable --now my-python-app
```

---

# 8. Logs anzeigen

`systemd` sammelt standardmäßig `stdout` und `stderr` des Prozesses.

Dadurch werden unter anderem normale Python-Ausgaben sichtbar:

```python
print("Application started")
```

## Gesamtes Log

```bash
sudo journalctl -u my-python-app
```

---

## Letzte 50 Einträge

```bash
sudo journalctl -u my-python-app -n 50
```

---

## Letzte 100 Einträge

```bash
sudo journalctl -u my-python-app -n 100
```

---

## Live-Log

Besonders praktisch während der Entwicklung:

```bash
sudo journalctl -u my-python-app -f
```

Mit:

```text
Ctrl+C
```

wird nur die Log-Anzeige beendet. Der Service selbst läuft weiter.

---

## Logs seit dem letzten Boot

```bash
sudo journalctl -u my-python-app -b
```

---

## Logs seit heute

```bash
sudo journalctl -u my-python-app --since today
```

---

## Typischer Debugging-Workflow

Terminal 1:

```bash
sudo journalctl -u my-python-app -f
```

Terminal 2:

```bash
sudo systemctl restart my-python-app
```

Dadurch sind Startfehler unmittelbar sichtbar.

---

# 9. User-Service als Alternative

Wenn der Service keinem systemweiten Dienst entsprechen soll, kann ein User-Service verwendet werden.

## 9.1 Verzeichnis erstellen

```bash
mkdir -p ~/.config/systemd/user
```

---

## 9.2 Service-Datei erstellen

```bash
nano ~/.config/systemd/user/my-python-app.service
```

Beispiel:

```ini
[Unit]
Description=My Python Application
After=network-online.target
Wants=network-online.target

[Service]
Type=simple

WorkingDirectory=/home/myuser/apps/my-python-app

ExecStart=/home/myuser/apps/my-python-app/.venv/bin/python /home/myuser/apps/my-python-app/src/main.py

Restart=on-failure
RestartSec=5

Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=default.target
```

Bei User-Services sind normalerweise keine `User=`- und `Group=`-Einträge notwendig.

---

## 9.3 Konfiguration laden

```bash
systemctl --user daemon-reload
```

---

## 9.4 Service aktivieren

```bash
systemctl --user enable my-python-app
```

---

## 9.5 Starten

```bash
systemctl --user start my-python-app
```

Oder kombiniert:

```bash
systemctl --user enable --now my-python-app
```

---

## 9.6 Verwalten

```bash
systemctl --user status my-python-app
```

```bash
systemctl --user restart my-python-app
```

```bash
systemctl --user stop my-python-app
```

---

## 9.7 Logs

```bash
journalctl --user -u my-python-app
```

Live:

```bash
journalctl --user -u my-python-app -f
```

---

# 10. User-Service bereits beim Booten starten

Standardmäßig hängt ein User-Service vom Benutzerkontext ab.

Soll der User-Service bereits beim Booten laufen, auch wenn sich der Benutzer noch nicht angemeldet hat, kann `linger` aktiviert werden.

```bash
sudo loginctl enable-linger myuser
```

Danach kann der User-Service unabhängig von einer interaktiven Anmeldung gestartet werden.

Status prüfen:

```bash
loginctl show-user myuser
```

Dort sollte unter anderem stehen:

```text
Linger=yes
```

Linger wieder deaktivieren:

```bash
sudo loginctl disable-linger myuser
```

Für klassische Serveranwendungen ist trotzdem häufig ein System-Service unter `/etc/systemd/system/` die klarere Lösung.

---

# 11. Besonderheiten unter Raspberry Pi OS

Raspberry Pi OS verwendet ebenfalls `systemd`. Die grundlegende Einrichtung unterscheidet sich daher kaum von Ubuntu.

## 11.1 Aktuelle Raspberry-Pi-OS-Version

Stand September 2026 basiert das aktuelle Raspberry Pi OS auf:

```text
Debian 13 "Trixie"
```

Der frühere Name **Raspbian** wird offiziell nicht mehr als Produktname verwendet. Das Betriebssystem heißt heute **Raspberry Pi OS**.

Die Anleitung funktioniert ebenfalls mit älteren Raspberry-Pi-OS-Versionen wie Debian 12 "Bookworm".

---

## 11.2 Python und virtuelle Umgebungen

Python 3 ist auf Raspberry Pi OS normalerweise bereits installiert.

Prüfen:

```bash
python3 --version
```

Bei aktuellen Versionen sollte für mit `pip` installierte Drittanbieter-Pakete eine virtuelle Umgebung verwendet werden:

```bash
python3 -m venv .venv
```

Dann:

```bash
source .venv/bin/activate
```

und:

```bash
pip install -r requirements.txt
```

Für einen Service wird wieder direkt der Interpreter aus der Umgebung verwendet:

```ini
ExecStart=/home/myuser/apps/my-python-app/.venv/bin/python /home/myuser/apps/my-python-app/src/main.py
```

---

## 11.3 Raspberry Pi OS Lite

Für einen Raspberry Pi, der ausschließlich als Server oder Controller verwendet wird, ist Raspberry Pi OS Lite häufig besonders geeignet.

Es wird keine Desktop-Oberfläche benötigt, um einen `systemd`-Service zu betreiben.

Ein typischer Aufbau ist:

```text
Raspberry Pi
    │
    ├── Raspberry Pi OS Lite
    │
    ├── systemd
    │    └── my-python-app.service
    │
    └── Python-Anwendung
         └── .venv
```

---

## 11.4 `sudo` auf aktuellen Raspberry-Pi-OS-Versionen

Auf aktuellen Raspberry-Pi-OS-Versionen ist passwortloses `sudo` nicht mehr grundsätzlich standardmäßig aktiviert.

Daher kann beispielsweise:

```bash
sudo systemctl restart my-python-app
```

nach dem Benutzerpasswort fragen.

Das ist normales und gewünschtes Sicherheitsverhalten.

---

## 11.5 Zugriff auf GPIO, I²C, SPI oder serielle Schnittstellen

Python-Anwendungen auf Raspberry Pis greifen häufig auf Hardware zu.

Beispiele:

- GPIO
- I²C
- SPI
- UART
- USB-Seriell

Die Anwendung sollte trotzdem möglichst unter einem normalen Benutzer laufen.

Prüfen, in welchen Gruppen der Benutzer ist:

```bash
groups myuser
```

Je nach Hardware können beispielsweise Gruppen wie diese relevant sein:

```text
gpio
i2c
spi
dialout
```

Beispiel für eine serielle Schnittstelle:

```bash
sudo usermod -aG dialout myuser
```

Für I²C:

```bash
sudo usermod -aG i2c myuser
```

Nach einer Änderung der Gruppenmitgliedschaft ist normalerweise eine neue Anmeldung oder ein Neustart sinnvoll.

> Die benötigten Gruppen hängen von der konkreten Hardware und der Raspberry-Pi-OS-Konfiguration ab.

---

## 11.6 Netzwerkdienste auf dem Raspberry Pi

Bei einer Anwendung, die nach dem Booten Netzwerkzugriff benötigt, ist folgende Konfiguration sinnvoll:

```ini
[Unit]
After=network-online.target
Wants=network-online.target
```

Wichtig ist trotzdem, dass Netzwerksoftware mit vorübergehend nicht verfügbarem Netzwerk umgehen kann.

Beispielsweise sollte eine Client-Anwendung Verbindungen erneut versuchen, statt dauerhaft abzustürzen.

---

## 11.7 Raspberry Pi neu starten und Service prüfen

Nach der Einrichtung:

```bash
sudo reboot
```

Nach dem Neustart:

```bash
sudo systemctl status my-python-app
```

Logs des aktuellen Boots:

```bash
sudo journalctl -u my-python-app -b
```

Damit lässt sich prüfen, ob die Anwendung tatsächlich automatisch gestartet wurde.

---

# 12. Umgebungsvariablen verwenden

Passwörter, Ports, Konfigurationspfade oder andere Einstellungen sollten nicht zwingend direkt in den Python-Code geschrieben werden.

## 12.1 Direkt in der Service-Datei

Beispiel:

```ini
Environment=APP_PORT=8080
Environment=APP_MODE=production
```

Python:

```python
import os

port = os.getenv("APP_PORT", "8080")
mode = os.getenv("APP_MODE", "development")
```

---

## 12.2 Separate Environment-Datei

Für größere Konfigurationen ist eine eigene Datei übersichtlicher.

Beispiel:

```bash
sudo nano /etc/my-python-app.env
```

Inhalt:

```text
APP_PORT=8080
APP_MODE=production
```

Service-Datei:

```ini
EnvironmentFile=/etc/my-python-app.env
```

Danach:

```bash
sudo systemctl daemon-reload
sudo systemctl restart my-python-app
```

> Dateien mit Passwörtern oder API-Schlüsseln sollten geeignete Dateirechte besitzen.

Beispielsweise:

```bash
sudo chmod 600 /etc/my-python-app.env
```

---

# 13. Anwendung nach einem Update neu starten

Angenommen, die Anwendung wird über Git aktualisiert.

Service stoppen:

```bash
sudo systemctl stop my-python-app
```

Projekt öffnen:

```bash
cd /home/myuser/apps/my-python-app
```

Aktualisieren:

```bash
git pull
```

Falls sich `requirements.txt` geändert hat:

```bash
/home/myuser/apps/my-python-app/.venv/bin/pip install -r requirements.txt
```

Service wieder starten:

```bash
sudo systemctl start my-python-app
```

Oder für einfache Code-Änderungen direkt:

```bash
sudo systemctl restart my-python-app
```

Danach kontrollieren:

```bash
sudo systemctl status my-python-app
```

und:

```bash
sudo journalctl -u my-python-app -n 50
```

---

# 14. Service wieder entfernen

Zuerst stoppen und deaktivieren:

```bash
sudo systemctl disable --now my-python-app
```

Service-Datei löschen:

```bash
sudo rm /etc/systemd/system/my-python-app.service
```

Danach:

```bash
sudo systemctl daemon-reload
```

Optional:

```bash
sudo systemctl reset-failed
```

---

# 15. Fehlersuche

## Problem: Service startet nicht

Status prüfen:

```bash
sudo systemctl status my-python-app
```

Logs:

```bash
sudo journalctl -u my-python-app -n 100
```

---

## Problem: `No such file or directory`

Meist ist ein Pfad in `ExecStart` falsch.

Prüfen:

```bash
ls -l /home/myuser/apps/my-python-app/.venv/bin/python
```

und:

```bash
ls -l /home/myuser/apps/my-python-app/src/main.py
```

Absolute Pfade verwenden.

---

## Problem: Python-Modul wird nicht gefunden

Zuerst denselben Befehl manuell ausführen:

```bash
cd /home/myuser/apps/my-python-app
/home/myuser/apps/my-python-app/.venv/bin/python src/main.py
```

Bei Python-Packages kann stattdessen ein Modulstart notwendig sein:

```bash
/home/myuser/apps/my-python-app/.venv/bin/python -m package_name.main
```

---

## Problem: Relative Dateien werden nicht gefunden

`WorkingDirectory` kontrollieren:

```ini
WorkingDirectory=/home/myuser/apps/my-python-app
```

Noch robuster ist es, Pfade innerhalb der Python-Anwendung auf Basis von `__file__` oder einer zentralen Pfadkonfiguration zu erzeugen.

---

## Problem: Service läuft manuell, aber nicht beim Booten

Prüfen:

```bash
sudo systemctl is-enabled my-python-app
```

Erwartete Ausgabe:

```text
enabled
```

Falls nicht:

```bash
sudo systemctl enable my-python-app
```

Logs des letzten Boots prüfen:

```bash
sudo journalctl -u my-python-app -b
```

---

## Problem: Netzwerk ist beim Start noch nicht verfügbar

Service:

```ini
After=network-online.target
Wants=network-online.target
```

verwenden.

Zusätzlich sollte die Python-Anwendung Netzwerkfehler selbst behandeln und Verbindungsversuche wiederholen.

---

## Problem: Anwendung startet ständig neu

Service-Status ansehen:

```bash
sudo systemctl status my-python-app
```

Logs:

```bash
sudo journalctl -u my-python-app -f
```

Bei:

```ini
Restart=on-failure
RestartSec=5
```

weist ein Neustart alle fünf Sekunden typischerweise darauf hin, dass die Python-Anwendung direkt nach dem Start einen Fehler produziert.

---

## Problem: `print()` erscheint verspätet im Log

Diese Zeile ergänzen:

```ini
Environment=PYTHONUNBUFFERED=1
```

Alternativ Python mit `-u` starten:

```ini
ExecStart=/home/myuser/apps/my-python-app/.venv/bin/python -u /home/myuser/apps/my-python-app/src/main.py
```

---

## Problem: Fehlende Zugriffsrechte

Der in:

```ini
User=myuser
```

angegebene Benutzer muss auf alle benötigten Dateien und Geräte zugreifen dürfen.

Dateien prüfen:

```bash
ls -la /home/myuser/apps/my-python-app
```

Benutzergruppen prüfen:

```bash
groups myuser
```

---

# 16. Empfohlene Konfiguration

Für eine typische dauerhaft laufende Python-Serveranwendung unter Ubuntu oder Raspberry Pi OS ist folgende Konfiguration ein guter Ausgangspunkt:

```ini
[Unit]
Description=My Python Application
After=network-online.target
Wants=network-online.target

[Service]
Type=simple

User=myuser
Group=myuser

WorkingDirectory=/home/myuser/apps/my-python-app

ExecStart=/home/myuser/apps/my-python-app/.venv/bin/python /home/myuser/apps/my-python-app/src/main.py

Restart=on-failure
RestartSec=5

Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

Danach:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now my-python-app
```

Status:

```bash
sudo systemctl status my-python-app
```

Live-Logs:

```bash
sudo journalctl -u my-python-app -f
```

---

# 17. Befehlsübersicht

| Aufgabe | System-Service |
|---|---|
| Starten | `sudo systemctl start my-python-app` |
| Stoppen | `sudo systemctl stop my-python-app` |
| Neustarten | `sudo systemctl restart my-python-app` |
| Status | `sudo systemctl status my-python-app` |
| Autostart aktivieren | `sudo systemctl enable my-python-app` |
| Sofort starten + Autostart | `sudo systemctl enable --now my-python-app` |
| Autostart deaktivieren | `sudo systemctl disable my-python-app` |
| Stoppen + deaktivieren | `sudo systemctl disable --now my-python-app` |
| Live-Log | `sudo journalctl -u my-python-app -f` |
| Letzte 50 Logs | `sudo journalctl -u my-python-app -n 50` |
| Logs dieses Boots | `sudo journalctl -u my-python-app -b` |
| Service-Datei neu laden | `sudo systemctl daemon-reload` |

Für einen User-Service:

| Aufgabe | User-Service |
|---|---|
| Starten | `systemctl --user start my-python-app` |
| Stoppen | `systemctl --user stop my-python-app` |
| Neustarten | `systemctl --user restart my-python-app` |
| Status | `systemctl --user status my-python-app` |
| Autostart aktivieren | `systemctl --user enable my-python-app` |
| Sofort starten + Autostart | `systemctl --user enable --now my-python-app` |
| Live-Log | `journalctl --user -u my-python-app -f` |
| Konfiguration neu laden | `systemctl --user daemon-reload` |

---

# Kurzfassung

Für eine Python-Anwendung, die dauerhaft auf einem Ubuntu- oder Raspberry-Pi-OS-System laufen soll:

1. Projekt in einen festen Ordner legen.
2. Virtuelle Python-Umgebung erstellen.
3. Abhängigkeiten in der `venv` installieren.
4. Startbefehl manuell testen.
5. `/etc/systemd/system/my-python-app.service` erstellen.
6. `systemctl daemon-reload` ausführen.
7. Service mit `systemctl enable --now` aktivieren.
8. Status mit `systemctl status` prüfen.
9. Logs mit `journalctl` überwachen.

Damit läuft die Python-Anwendung unabhängig von einer geöffneten Konsole und kann nach einem Systemneustart automatisch wieder gestartet werden.

---

## Hinweise zum Stand der Anleitung

Stand: **September 2026**

- Das aktuelle Raspberry Pi OS basiert auf **Debian 13 (Trixie)**.
- Raspberry Pi OS hieß früher „Raspbian“.
- Für über `pip` installierte Python-Pakete werden auf aktuellen Raspberry-Pi-OS-Versionen virtuelle Umgebungen empfohlen bzw. durch Debians Schutz des System-Python vorausgesetzt.
- Aktuelle Raspberry-Pi-OS-Versionen verwenden `systemd`.
- Passwortloses `sudo` ist auf aktuellen Raspberry-Pi-OS-Installationen nicht mehr standardmäßig aktiviert.

### Weiterführende offizielle Dokumentation

- Raspberry Pi Documentation: **Raspberry Pi OS**
- Raspberry Pi Documentation: **Using Python with virtual environments**
- Raspberry Pi OS Downloads / Release Information
- systemd Documentation: **systemd.service**
- systemd Documentation: **systemctl**
- systemd Documentation: **loginctl**
