# Python installieren und Virtual Environment einrichten

Diese Anleitung beschreibt Schritt für Schritt, wie Python unter **Linux (Ubuntu/Debian/Raspberry Pi OS)** und **Windows** installiert wird und wie anschließend innerhalb eines Projektordners eine Python Virtual Environment (`venv`) eingerichtet und verwendet wird.

---

# Inhaltsverzeichnis

1. [Warum eine Virtual Environment?](#1-warum-eine-virtual-environment)
2. [Linux: Python installieren](#2-linux-python-installieren)
3. [Linux: Virtual Environment erstellen](#3-linux-virtual-environment-erstellen)
4. [Windows: Python installieren](#4-windows-python-installieren)
5. [Windows: Virtual Environment erstellen](#5-windows-virtual-environment-erstellen)
6. [Pakete installieren](#6-pakete-installieren)
7. [Bestehende Projekte einrichten](#7-bestehende-projekte-einrichten)
8. [GitHub-Projekt einrichten](#8-github-projekt-einrichten)
9. [Virtual Environment beenden und erneut aktivieren](#9-virtual-environment-beenden-und-erneut-aktivieren)
10. [`.gitignore`](#10-gitignore)
11. [Typische Fehler](#11-typische-fehler)
12. [Kurzreferenz](#12-kurzreferenz)

---

# 1. Warum eine Virtual Environment?

Eine Virtual Environment ist eine isolierte Python-Umgebung für ein einzelnes Projekt.

Dadurch können unterschiedliche Projekte unterschiedliche Versionen von Python-Paketen verwenden, ohne sich gegenseitig zu beeinflussen.

Beispiel:

```text
Projekt A
└── requests 2.31

Projekt B
└── requests 2.32
```

Die virtuelle Umgebung wird normalerweise direkt im Projektordner angelegt:

```text
mein-projekt/
├── .venv/
├── src/
├── requirements.txt
├── pyproject.toml
└── README.md
```

Empfehlung:

```text
.venv
```

als Name für die Virtual Environment verwenden.

---

# 2. Linux: Python installieren

Diese Schritte gelten insbesondere für:

- Ubuntu
- Debian
- Raspberry Pi OS
- Linux Mint
- andere Debian-basierte Systeme

## 2.1 Prüfen, ob Python bereits installiert ist

Terminal öffnen und ausführen:

```bash
python3 --version
```

Beispiel:

```text
Python 3.12.3
```

Zusätzlich kann geprüft werden, wo Python installiert ist:

```bash
which python3
```

Typische Ausgabe:

```text
/usr/bin/python3
```

---

## 2.2 Paketlisten aktualisieren

```bash
sudo apt update
```

Optional können auch bereits installierte Pakete aktualisiert werden:

```bash
sudo apt upgrade
```

---

## 2.3 Python, pip und venv installieren

```bash
sudo apt install python3 python3-pip python3-venv
```

Anschließend prüfen:

```bash
python3 --version
```

und:

```bash
pip3 --version
```

---

# 3. Linux: Virtual Environment erstellen

## 3.1 In den Projektordner wechseln

Beispiel:

```bash
cd ~/projects/mein-projekt
```

Aktuellen Ordner anzeigen:

```bash
pwd
```

---

## 3.2 Virtual Environment erstellen

```bash
python3 -m venv .venv
```

Danach befindet sich im Projektordner ein neuer Ordner:

```text
.venv/
```

---

## 3.3 Virtual Environment aktivieren

```bash
source .venv/bin/activate
```

Das Terminal zeigt anschließend normalerweise:

```text
(.venv) user@linux:~/projects/mein-projekt$
```

---

## 3.4 Prüfen, welches Python verwendet wird

```bash
which python
```

Die Ausgabe sollte auf die Virtual Environment zeigen, zum Beispiel:

```text
/home/user/projects/mein-projekt/.venv/bin/python
```

Zusätzlich:

```bash
python --version
```

und:

```bash
pip --version
```

---

# 4. Windows: Python installieren

## 4.1 Prüfen, ob Python bereits installiert ist

PowerShell oder Eingabeaufforderung öffnen:

```powershell
python --version
```

Alternativ:

```powershell
py --version
```

Beispiel:

```text
Python 3.12.7
```

Falls Python nicht gefunden wird, muss es installiert werden.

---

## 4.2 Python installieren

Python kann über die offizielle Python-Webseite installiert werden.

Während der Installation unbedingt folgende Option aktivieren:

```text
Add Python to PATH
```

Danach empfiehlt es sich, PowerShell oder die Eingabeaufforderung neu zu öffnen.

Prüfen:

```powershell
python --version
```

oder:

```powershell
py --version
```

Zusätzlich:

```powershell
pip --version
```

---

## 4.3 Optional: Python Launcher verwenden

Unter Windows ist der Python Launcher `py` häufig die robusteste Variante.

Zum Beispiel:

```powershell
py -3 --version
```

oder:

```powershell
py -3.12 --version
```

Eine Virtual Environment kann deshalb auch so erstellt werden:

```powershell
py -m venv .venv
```

---

# 5. Windows: Virtual Environment erstellen

## 5.1 In den Projektordner wechseln

Beispiel:

```powershell
cd C:\Users\MeinBenutzerName\Projects\mein-projekt
```

Aktuellen Ordner anzeigen:

```powershell
Get-Location
```

In der klassischen Eingabeaufforderung:

```cmd
cd
```

---

## 5.2 Virtual Environment erstellen

Empfohlen:

```powershell
python -m venv .venv
```

Alternativ:

```powershell
py -m venv .venv
```

Danach sieht der Projektordner beispielsweise so aus:

```text
mein-projekt/
├── .venv/
├── src/
├── requirements.txt
├── pyproject.toml
└── README.md
```

---

## 5.3 Virtual Environment in PowerShell aktivieren

```powershell
.\.venv\Scripts\Activate.ps1
```

Danach sollte vor der Eingabeaufforderung stehen:

```text
(.venv)
```

Beispiel:

```text
(.venv) PS C:\Users\MeinBenutzerName\Projects\mein-projekt>
```

---

## 5.4 Virtual Environment in CMD aktivieren

Falls die klassische Eingabeaufforderung verwendet wird:

```cmd
.venv\Scripts\activate.bat
```

---

## 5.5 Virtual Environment in Git Bash aktivieren

```bash
source .venv/Scripts/activate
```

---

## 5.6 PowerShell blockiert `Activate.ps1`

Eine häufige Fehlermeldung lautet sinngemäß:

```text
running scripts is disabled on this system
```

Dann kann für den aktuellen Benutzer die Ausführungsrichtlinie angepasst werden:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

PowerShell fragt normalerweise nach einer Bestätigung.

Danach erneut:

```powershell
.\.venv\Scripts\Activate.ps1
```

Die Änderung gilt nur für den aktuellen Windows-Benutzer.

---

## 5.7 Prüfen, welches Python verwendet wird

```powershell
where.exe python
```

Die erste Ausgabe sollte auf die Virtual Environment zeigen, zum Beispiel:

```text
C:\Users\MeinBenutzerName\Projects\mein-projekt\.venv\Scripts\python.exe
```

Zusätzlich:

```powershell
python --version
```

und:

```powershell
pip --version
```

---

# 6. Pakete installieren

Sobald die Virtual Environment aktiviert ist, sollte zunächst `pip` aktualisiert werden:

```bash
python -m pip install --upgrade pip
```

Das funktioniert unter Linux und Windows gleich.

Ein einzelnes Paket installieren:

```bash
pip install requests
```

Mehrere Pakete:

```bash
pip install requests PyQt6 numpy
```

Installierte Pakete anzeigen:

```bash
pip list
```

---

# 7. Bestehende Projekte einrichten

## 7.1 Projekt mit `requirements.txt`

Falls das Projekt eine Datei

```text
requirements.txt
```

enthält:

```bash
pip install -r requirements.txt
```

---

## 7.2 Projekt mit `pyproject.toml`

Wenn das Projekt als Python-Package aufgebaut ist und eine `pyproject.toml` besitzt:

```bash
pip install -e .
```

Das `-e` steht für **editable installation**.

Das bedeutet:

```text
Quellcode ändern
        ↓
keine erneute Installation erforderlich
        ↓
Änderungen werden direkt verwendet
```

Eine erneute Installation ist normalerweise nur notwendig, wenn sich beispielsweise die Dependencies in `pyproject.toml` ändern.

---

## 7.3 Programm starten

Ein einzelnes Python-File:

```bash
python main.py
```

Ein Python-Package:

```bash
python -m mein_package
```

Beispiel:

```bash
python -m yrail_backend
```

---

# 8. GitHub-Projekt einrichten

Ein typischer Workflow für ein neu geklontes Python-Projekt sieht so aus.

## Linux

```bash
cd ~/projects

git clone git@github.com:USERNAME/REPOSITORY.git

cd REPOSITORY

python3 -m venv .venv

source .venv/bin/activate

python -m pip install --upgrade pip
```

Danach je nach Projekt:

```bash
pip install -r requirements.txt
```

oder:

```bash
pip install -e .
```

---

## Windows PowerShell

```powershell
cd C:\Users\MeinBenutzerName\Projects

git clone git@github.com:USERNAME/REPOSITORY.git

cd REPOSITORY

py -m venv .venv

.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
```

Danach:

```powershell
pip install -r requirements.txt
```

oder:

```powershell
pip install -e .
```

---

# 9. Virtual Environment beenden und erneut aktivieren

## Virtual Environment verlassen

Unter Windows und Linux:

```bash
deactivate
```

Die Anzeige

```text
(.venv)
```

verschwindet anschließend.

---

## Beim nächsten Arbeiten am Projekt

Die Virtual Environment muss **nicht neu erstellt** werden.

### Linux

```bash
cd ~/projects/mein-projekt
source .venv/bin/activate
```

### Windows PowerShell

```powershell
cd C:\Users\MeinBenutzerName\Projects\mein-projekt
.\.venv\Scripts\Activate.ps1
```

### Windows CMD

```cmd
cd C:\Users\MeinBenutzerName\Projects\mein-projekt
.venv\Scripts\activate.bat
```

Danach kann das Projekt direkt gestartet werden.

---

# 10. `.gitignore`

Die Virtual Environment darf normalerweise **nicht in Git eingecheckt** werden.

Deshalb sollte in der `.gitignore` stehen:

```gitignore
.venv/
```

Zusätzlich sind für Python-Projekte häufig sinnvoll:

```gitignore
.venv/
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.pytest_cache/
.mypy_cache/
```

Die Abhängigkeiten werden stattdessen über Dateien wie diese beschrieben:

```text
requirements.txt
```

oder:

```text
pyproject.toml
```

---

# 11. Typische Fehler

## `python: command not found` unter Linux

Versuche:

```bash
python3 --version
```

Außerhalb einer Virtual Environment ist unter Linux häufig nur `python3` verfügbar.

---

## `python is not recognized` unter Windows

Mögliche Ursachen:

- Python ist nicht installiert.
- Python wurde nicht zum `PATH` hinzugefügt.
- Das Terminal wurde nach der Installation nicht neu geöffnet.

Versuche:

```powershell
py --version
```

Wenn `py` funktioniert, kann die venv so erstellt werden:

```powershell
py -m venv .venv
```

---

## `No module named venv` unter Linux

Installieren:

```bash
sudo apt install python3-venv
```

Danach:

```bash
python3 -m venv .venv
```

---

## `pip` installiert Pakete global

Zuerst prüfen, ob die Virtual Environment aktiv ist.

Linux:

```bash
which python
which pip
```

Windows:

```powershell
where.exe python
where.exe pip
```

Beide Befehle sollten zuerst auf den `.venv`-Ordner zeigen.

---

## PowerShell erlaubt keine Skriptausführung

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Danach erneut:

```powershell
.\.venv\Scripts\Activate.ps1
```

---

## Virtual Environment wurde verschoben

Eine Virtual Environment sollte nach Möglichkeit **nicht zwischen Ordnern oder Rechnern verschoben** werden.

Besser:

```text
alte .venv löschen
        ↓
neue .venv erstellen
        ↓
Dependencies neu installieren
```

Linux:

```bash
rm -rf .venv
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Windows PowerShell:

```powershell
Remove-Item -Recurse -Force .venv
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

---

# 12. Kurzreferenz

## Linux

Neue Umgebung:

```bash
cd ~/projects/mein-projekt
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

Bestehende Umgebung aktivieren:

```bash
source .venv/bin/activate
```

Umgebung verlassen:

```bash
deactivate
```

---

## Windows PowerShell

Neue Umgebung:

```powershell
cd C:\Users\MeinBenutzerName\Projects\mein-projekt
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

Bestehende Umgebung aktivieren:

```powershell
.\.venv\Scripts\Activate.ps1
```

Umgebung verlassen:

```powershell
deactivate
```

---

# Empfohlener Standard-Workflow

Für praktisch jedes neue Python-Projekt:

```text
1. Repository klonen oder Projektordner anlegen
2. In den Projektordner wechseln
3. .venv erstellen
4. .venv aktivieren
5. pip aktualisieren
6. Dependencies installieren
7. Projekt starten
```

Die `.venv` bleibt lokal auf dem jeweiligen Rechner und wird nicht mit Git synchronisiert.
