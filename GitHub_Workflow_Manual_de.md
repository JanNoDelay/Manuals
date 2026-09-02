# Git & GitHub Workflow -- Komplette Anleitung

> Eine praxisorientierte Schritt-für-Schritt-Anleitung für die
> Zusammenarbeit über GitHub unter Windows und Ubuntu/Linux -- mit SSH,
> Branches, Pull Requests und Code Reviews.

## 1. Grundprinzip

Git verwaltet die Versionshistorie **lokal** auf deinem Rechner. GitHub
ist das **Remote-Repository**, über das du mit anderen Entwicklern
zusammenarbeitest.

Ein typischer Ablauf ist:

``` text
GitHub Repository
      │
      │ git clone / git pull
      ▼
Lokales Repository
      │
      ├── Dateien bearbeiten
      │
      ├── git add
      │
      ├── git commit
      │
      └── git push
      ▼
GitHub Repository
```

Bei professioneller Zusammenarbeit empfiehlt sich meistens:

``` text
main
 │
 └── Feature-Branch
        │
        ├── Änderungen
        ├── Commits
        └── Push
             │
             ▼
        Pull Request
             │
             ▼
        Code Review
             │
             ▼
           Merge
             │
             ▼
            main
```

------------------------------------------------------------------------

# 2. Git installieren

## Windows

Git for Windows installieren:

-   https://git-scm.com/download/win

Danach in PowerShell oder Git Bash prüfen:

``` powershell
git --version
```

## Ubuntu / Linux

``` bash
sudo apt update
sudo apt install git openssh-client
```

Prüfen:

``` bash
git --version
ssh -V
```

------------------------------------------------------------------------

# 3. Git-Benutzer konfigurieren

Git speichert bei jedem Commit einen Namen und eine E-Mail-Adresse.

``` bash
git config --global user.name "Dein Name"
git config --global user.email "deine-email@example.com"
```

Konfiguration anzeigen:

``` bash
git config --global --list
```

Oder gezielt:

``` bash
git config --global user.name
git config --global user.email
```

## Benutzer ändern

Vorhandene Werte können einfach überschrieben werden:

``` bash
git config --global user.name "Neuer Name"
git config --global user.email "neue-email@example.com"
```

Alternativ zuerst entfernen:

``` bash
git config --global --unset user.name
git config --global --unset user.email
```

## Nur für ein bestimmtes Repository

Innerhalb des Repository-Ordners:

``` bash
git config user.name "Name für dieses Projekt"
git config user.email "projekt-email@example.com"
```

Ohne `--global` gilt die Einstellung nur für dieses Repository.

## Herkunft einer Einstellung prüfen

``` bash
git config --list --show-origin
```

Das ist besonders hilfreich, wenn dieselbe Einstellung lokal, global
oder systemweit vorkommt.

## Einzelnen Credential-Eintrag entfernen

Beispiel:

``` text
credential.https://git.acdp.at.provider=generic
```

Global entfernen:

``` bash
git config --global --unset credential.https://git.acdp.at.provider
```

Falls der Eintrag nur im aktuellen Repository liegt:

``` bash
git config --unset credential.https://git.acdp.at.provider
```

Systemweit:

``` bash
sudo git config --system --unset credential.https://git.acdp.at.provider
```

------------------------------------------------------------------------

# 4. GitHub über SSH einrichten

GitHub akzeptiert bei Git-Operationen über HTTPS kein normales
Account-Passwort mehr. Für Entwicklungsrechner ist SSH eine komfortable
Lösung.

## 4.1 SSH-Key unter Ubuntu/Linux erstellen

Standardordner prüfen:

``` bash
ls -la ~/.ssh
```

Falls nötig:

``` bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Key erstellen:

``` bash
ssh-keygen -t ed25519 -C "deine-email@example.com"
```

Bei:

``` text
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
```

einfach **Enter** drücken.

Dadurch entstehen:

``` text
~/.ssh/id_ed25519       # privater Key
~/.ssh/id_ed25519.pub   # öffentlicher Key
```

Der private Key darf **niemals weitergegeben** werden.

## 4.2 SSH-Agent unter Linux starten

``` bash
eval "$(ssh-agent -s)"
```

Key laden:

``` bash
ssh-add ~/.ssh/id_ed25519
```

Prüfen:

``` bash
ssh-add -l
```

Public Key anzeigen:

``` bash
cat ~/.ssh/id_ed25519.pub
```

Die komplette ausgegebene Zeile kopieren.

> Hinweis: Wenn der SSH-Agent nur für die aktuelle Shell gestartet
> wurde, kann nach dem Schließen des Terminals ein erneutes `ssh-add`
> bzw. Starten des Agents nötig sein.

## 4.3 SSH-Key unter Windows erstellen

In PowerShell:

``` powershell
ssh-keygen -t ed25519 -C "deine-email@example.com"
```

Beim Speicherort Enter drücken. Standardmäßig:

``` text
C:\Users\<Benutzername>\.ssh\id_ed25519
C:\Users\<Benutzername>\.ssh\id_ed25519.pub
```

Public Key anzeigen:

``` powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

SSH-Agent prüfen:

``` powershell
Get-Service ssh-agent
```

Falls nötig starten:

``` powershell
Start-Service ssh-agent
```

Key hinzufügen:

``` powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

## 4.4 Public Key bei GitHub hinterlegen

Auf GitHub:

1.  **Settings**
2.  **SSH and GPG keys**
3.  **New SSH key**
4.  Einen Namen vergeben, z. B. `Ubuntu Development PC`
5.  Inhalt von `id_ed25519.pub` einfügen
6.  Speichern

## 4.5 SSH-Verbindung testen

``` bash
ssh -T git@github.com
```

Bei erfolgreicher Authentifizierung erscheint sinngemäß:

``` text
Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

Beim ersten Verbindungsaufbau kann SSH nach der Vertrauenswürdigkeit des
Hosts fragen. Nach Prüfung des GitHub-Hostkeys kann dieser bestätigt
werden.

------------------------------------------------------------------------

# 5. Fehler: Permission denied (publickey)

Wenn Folgendes erscheint:

``` text
git@github.com: Permission denied (publickey).
fatal: Konnte nicht vom Remote-Repository lesen.
```

zuerst prüfen:

``` bash
ssh-add -l
```

Falls kein Key geladen ist:

``` bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Danach:

``` bash
ssh -T git@github.com
```

Für detaillierte Fehlersuche:

``` bash
ssh -vT git@github.com
```

Zusätzlich Remote prüfen:

``` bash
git remote -v
```

Für GitHub über SSH sollte die Adresse ungefähr so aussehen:

``` text
git@github.com:Organisation/Repository.git
```

Nicht `sudo git push` verwenden. `sudo` kann dazu führen, dass SSH unter
einem anderen Benutzer und damit mit anderen Keys ausgeführt wird.

------------------------------------------------------------------------

# 6. Repository das erste Mal klonen

Auf GitHub:

**Repository → Code → SSH**

Die SSH-Adresse sieht beispielsweise so aus:

``` text
git@github.com:Organisation/Projekt.git
```

## Ubuntu/Linux

``` bash
cd ~/Projekte
git clone git@github.com:Organisation/Projekt.git
cd Projekt
```

## Windows

``` powershell
cd D:\Projekte
git clone git@github.com:Organisation/Projekt.git
cd Projekt
```

Danach:

``` bash
git status
git remote -v
```

------------------------------------------------------------------------

# 7. Normaler Workflow auf `main`

Vor Beginn der Arbeit:

``` bash
git switch main
git pull
```

Dann Dateien bearbeiten.

Status prüfen:

``` bash
git status
```

Alle Änderungen stagen:

``` bash
git add .
```

Oder nur bestimmte Dateien:

``` bash
git add src/main.py
```

Noch einmal prüfen:

``` bash
git status
```

Commit erstellen:

``` bash
git commit -m "Fix communication handling"
```

Push:

``` bash
git push
```

Kompletter Ablauf:

``` bash
git switch main
git pull

# Dateien bearbeiten

git status
git add .
git status
git commit -m "Beschreibung der Änderungen"
git push
```

------------------------------------------------------------------------

# 8. `git commit -m` vs. `git commit -a`

## `git commit -m`

`-m` steht für **message**:

``` bash
git commit -m "Fix heartbeat timeout"
```

Es werden nur Änderungen committed, die vorher gestaged wurden.

Typisch:

``` bash
git add .
git commit -m "Fix heartbeat timeout"
```

## `git commit -a`

`-a` steht für **all tracked files**:

``` bash
git commit -a
```

Git staged automatisch Änderungen und Löschungen an **bereits getrackten
Dateien**.

Neue, noch untracked Dateien werden **nicht** hinzugefügt.

## Kombination

``` bash
git commit -am "Fix heartbeat handling"
```

Das ist praktisch für Änderungen ausschließlich an bereits bekannten
Dateien.

Für Kundenprojekte ist der explizite Ablauf meist übersichtlicher:

``` bash
git status
git add .
git status
git commit -m "Beschreibung"
```

So lässt sich vor dem Commit besser kontrollieren, was tatsächlich
enthalten ist.

------------------------------------------------------------------------

# 9. Branches verstehen

Ein Branch ist eine eigene Entwicklungslinie innerhalb desselben
Repositorys.

Wichtig: Beim Branch-Wechsel bleibt dein lokaler Pfad gleich.

Beispiel:

``` text
/home/user/Projekte/Projekt/
```

Nach:

``` bash
git switch feature/new-api
```

bleibst du im selben Ordner. Git passt lediglich die Dateien im Working
Tree an den gewählten Branch an.

Pfad prüfen:

``` bash
pwd
```

Aktuellen Branch anzeigen:

``` bash
git branch
```

Beispiel:

``` text
  main
* feature/new-api
```

------------------------------------------------------------------------

# 10. Neuen Branch erstellen, der noch nicht auf GitHub existiert

Zuerst `main` aktualisieren:

``` bash
git switch main
git pull
```

Dann neuen Branch erstellen und direkt wechseln:

``` bash
git switch -c feature/neues-feature
```

Prüfen:

``` bash
git branch
```

Dann Änderungen vornehmen und committen:

``` bash
git status
git add .
git commit -m "Implement new feature"
```

Da der Branch auf GitHub noch nicht existiert, beim **ersten Push**:

``` bash
git push -u origin feature/neues-feature
```

`-u` bzw. `--set-upstream` verbindet:

``` text
lokal:  feature/neues-feature
          ↕
remote: origin/feature/neues-feature
```

Ab jetzt reichen:

``` bash
git push
```

und:

``` bash
git pull
```

------------------------------------------------------------------------

# 11. Branch existiert bereits auf GitHub

Zuerst Remote-Informationen aktualisieren:

``` bash
git fetch
```

Alle Branches anzeigen:

``` bash
git branch -a
```

Beispiel:

``` text
* main
  remotes/origin/main
  remotes/origin/feature/new-api
```

Auf den vorhandenen Branch wechseln:

``` bash
git switch feature/new-api
```

Bei einer aktuellen Git-Version wird normalerweise automatisch ein
lokaler Tracking-Branch für `origin/feature/new-api` erstellt.

Danach:

``` bash
git status
git pull
```

Jetzt kann normal gearbeitet werden:

``` bash
git add .
git commit -m "Update API handling"
git push
```

Falls die automatische Zuordnung einmal nicht funktioniert:

``` bash
git switch --track origin/feature/new-api
```

------------------------------------------------------------------------

# 12. Zwischen Branches wechseln

Zu `main`:

``` bash
git switch main
```

Zu einem anderen Branch:

``` bash
git switch feature/neues-feature
```

Lokale Branches:

``` bash
git branch
```

Lokale und Remote-Branches:

``` bash
git branch -a
```

Nur Remote-Branches:

``` bash
git branch -r
```

Remote-Informationen aktualisieren:

``` bash
git fetch
```

------------------------------------------------------------------------

# 13. Vor einem Branch-Wechsel

Immer zuerst:

``` bash
git status
```

Optimal:

``` text
nothing to commit, working tree clean
```

Dann kann sicher gewechselt werden.

Sind noch Änderungen vorhanden, können sie committed werden:

``` bash
git add .
git commit -m "Save current work"
```

Oder temporär mit Stash abgelegt werden:

``` bash
git stash
git switch anderer-branch
```

Später:

``` bash
git stash pop
```

------------------------------------------------------------------------

# 14. Bereits auf `main` Änderungen gemacht -- trotzdem neuen Branch erstellen

Wenn du Änderungen gemacht, aber noch **nicht committed** hast, kannst
du normalerweise einen neuen Branch erstellen:

``` bash
git switch -c feature/meine-aenderung
```

Die lokalen Änderungen bleiben erhalten.

Danach:

``` bash
git add .
git commit -m "Implement feature"
git push -u origin feature/meine-aenderung
```

So landen die Änderungen sauber auf dem neuen Feature-Branch statt auf
`main`.

------------------------------------------------------------------------

# 15. Pull Request mit verpflichtendem Review

Angenommen, du hast lokal bereits:

``` bash
git add .
git commit -m "Implement feature"
```

ausgeführt.

## Schritt 1: Branch zu GitHub pushen

Beim ersten Push:

``` bash
git push -u origin feature/meine-aenderung
```

## Schritt 2: Pull Request erstellen

Auf GitHub:

**Pull requests → New pull request**

Auswahl:

``` text
base:    main
compare: feature/meine-aenderung
```

Bedeutung:

``` text
feature/meine-aenderung
          │
          ▼
         main
```

Pull Request mit aussagekräftigem Titel und Beschreibung erstellen.

## Schritt 3: Reviewer auswählen

Im Pull Request unter **Reviewers** den zuständigen Kollegen auswählen.

Der Workflow ist dann:

``` text
Feature-Branch
      │
      ▼
Pull Request
      │
      ▼
Code Review
      │
      ├── Changes requested
      │
      └── Approved
              │
              ▼
             Merge
              │
              ▼
             main
```

Repository-Regeln/Branch Protection können so konfiguriert sein, dass
ein Merge erst nach mindestens einem Approval erlaubt ist.

------------------------------------------------------------------------

# 16. Reviewer verlangt Änderungen

Es muss **kein neuer Pull Request** erstellt werden.

Auf demselben Branch weiterarbeiten:

``` bash
git switch feature/meine-aenderung
```

Änderungen durchführen:

``` bash
git status
git add .
git commit -m "Address review comments"
git push
```

Der bestehende Pull Request wird automatisch mit den neuen Commits
aktualisiert.

Ein Pull Request bezieht sich auf den Branch bzw. die Differenz zwischen
Branches, nicht nur auf einen einzelnen Commit.

------------------------------------------------------------------------

# 17. Pull Request mergen

Nach erfolgreichem Review kann der PR -- abhängig von den
Repository-Rechten und Regeln -- gemergt werden.

Typische GitHub-Optionen:

### Merge commit

Behält die Branch-/Commit-Struktur bei.

### Squash and merge

Mehrere Commits des Feature-Branches werden zu einem Commit auf `main`
zusammengefasst.

Das kann sinnvoll sein bei:

``` text
Add heartbeat
Fix timeout
Fix typo
Address review comments
Improve logging
```

Aus diesen Commits wird beispielsweise:

``` text
Implement frontend heartbeat handling
```

### Rebase and merge

Übernimmt die einzelnen Commits ohne zusätzlichen Merge-Commit in eine
lineare Historie.

Welche Strategie verwendet wird, sollte sich nach den Regeln des
Projekts richten.

------------------------------------------------------------------------

# 18. Nach dem Merge lokal aufräumen

Nach dem Merge:

``` bash
git switch main
git pull
```

Damit enthält dein lokaler `main` den neuen Stand.

Lokalen Feature-Branch löschen:

``` bash
git branch -d feature/meine-aenderung
```

Falls der Remote-Branch noch existiert:

``` bash
git push origin --delete feature/meine-aenderung
```

Veraltete Remote-Referenzen entfernen:

``` bash
git fetch --prune
```

------------------------------------------------------------------------

# 19. Empfohlener Workflow für Zusammenarbeit

Für Änderungen, die reviewed werden sollen:

``` bash
# 1. Aktuellen main holen
git switch main
git pull

# 2. Feature-Branch erstellen
git switch -c feature/meine-aenderung

# 3. Arbeiten

# 4. Änderungen kontrollieren und committen
git status
git add .
git status
git commit -m "Implement feature"

# 5. Branch erstmals pushen
git push -u origin feature/meine-aenderung
```

Dann auf GitHub:

``` text
Pull Request erstellen
        ↓
Reviewer auswählen
        ↓
Review
        ↓
ggf. Änderungen + weitere Commits
        ↓
Approval
        ↓
Merge nach main
```

Danach lokal:

``` bash
git switch main
git pull
git branch -d feature/meine-aenderung
git fetch --prune
```

------------------------------------------------------------------------

# 20. Direkter Workflow auf `main`

Nur verwenden, wenn das Projekt direkte Pushes auf `main` erlaubt und
dies im Team gewünscht ist:

``` bash
git switch main
git pull

# Änderungen

git status
git add .
git status
git commit -m "Beschreibung"
git push
```

Bei Team-/Kundenprojekten sind Feature-Branches und Pull Requests
meistens sicherer.

------------------------------------------------------------------------

# 21. Praktischer Spickzettel

  Aufgabe                       Befehl
  ----------------------------- ------------------------------------------
  Repository klonen             `git clone git@github.com:user/repo.git`
  Status prüfen                 `git status`
  Remote anzeigen               `git remote -v`
  Lokale Branches               `git branch`
  Alle Branches                 `git branch -a`
  Remote-Branches               `git branch -r`
  Remote aktualisieren          `git fetch`
  `main` aktualisieren          `git switch main` + `git pull`
  Neuen Branch erstellen        `git switch -c feature/name`
  Branch wechseln               `git switch branch-name`
  Remote-Branch auschecken      `git switch branch-name`
  Änderungen stagen             `git add .`
  Einzelne Datei stagen         `git add datei.py`
  Commit                        `git commit -m "Message"`
  Erster Push neuer Branch      `git push -u origin branch-name`
  Normaler Push                 `git push`
  Änderungen holen              `git pull`
  Änderungen temporär ablegen   `git stash`
  Stash zurückholen             `git stash pop`
  Lokalen Branch löschen        `git branch -d branch-name`
  Remote-Branch löschen         `git push origin --delete branch-name`
  Alte Remote-Refs entfernen    `git fetch --prune`
  Git-Konfiguration prüfen      `git config --list --show-origin`
  SSH-Verbindung testen         `ssh -T git@github.com`
  SSH-Debugging                 `ssh -vT git@github.com`

------------------------------------------------------------------------

# 22. Empfohlene Routine vor jedem Push

Gerade bei einem gemeinsam genutzten Repository:

``` bash
git status
```

Prüfen, ob du auf dem richtigen Branch bist:

``` bash
git branch
```

Optional Änderungen ansehen:

``` bash
git diff
```

Gestagte Änderungen ansehen:

``` bash
git diff --staged
```

Dann erst:

``` bash
git commit -m "Aussagekräftige Commit-Nachricht"
git push
```

------------------------------------------------------------------------

# 23. Die wichtigsten Regeln

1.  **Vor neuer Arbeit `main` aktualisieren.**

    ``` bash
    git switch main
    git pull
    ```

2.  **Für größere Änderungen einen eigenen Branch verwenden.**

    ``` bash
    git switch -c feature/name
    ```

3.  **Vor `git add .` mit `git status` prüfen, was verändert wurde.**

4.  **Vor dem Commit erneut kontrollieren, was gestaged ist.**

5.  **Aussagekräftige Commit-Nachrichten verwenden.**

6.  **Beim ersten Push eines neuen Branches `-u` verwenden.**

    ``` bash
    git push -u origin feature/name
    ```

7.  **Review-Änderungen einfach auf denselben Branch pushen.** Der Pull
    Request aktualisiert sich automatisch.

8.  **Nach einem Merge den lokalen `main` aktualisieren.**

    ``` bash
    git switch main
    git pull
    ```

9.  **Vor einem Branch-Wechsel `git status` prüfen.**

10. **Private SSH-Keys niemals weitergeben oder committen.**

------------------------------------------------------------------------

## Beispiel: Kompletter Feature-Lebenszyklus

``` bash
# Repository ist bereits geklont

git switch main
git pull

git switch -c feature/heartbeat

# Code bearbeiten

git status
git add .
git status
git commit -m "Implement heartbeat handling"

git push -u origin feature/heartbeat
```

Auf GitHub:

``` text
feature/heartbeat
       │
       ▼
Pull Request → main
       │
       ▼
Kollege reviewed
       │
       ├── Changes requested
       │       ↓
       │   lokal ändern
       │   git add .
       │   git commit -m "Address review comments"
       │   git push
       │
       ▼
Approved
       │
       ▼
Merge
```

Danach lokal:

``` bash
git switch main
git pull
git branch -d feature/heartbeat
git fetch --prune
```

Damit ist der komplette Entwicklungszyklus abgeschlossen.
