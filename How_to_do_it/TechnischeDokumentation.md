# SSH-Zugriff auf Linux- und Windows-Server mit SSH-Schlüsseln

Diese Anleitung beschreibt, wie ein Benutzer ein **SSH-Schlüsselpaar** erstellt und anschließend sicheren Zugriff auf einen **Linux-Server** oder einen **Windows-Server** erhält.  
Dieses Prinzip ist sowohl für lokale Server als auch für Server in **Cloud-Umgebungen** geeignet.

Der Zugriff erfolgt über **SSH-Schlüssel** und nicht über Passwörter.

---

## 1. Grundprinzip von SSH mit Schlüsseln

Die SSH-Authentifizierung funktioniert nach folgendem Prinzip:

1. Der **Client** erstellt ein **SSH-Schlüsselpaar**
2. Der **Private Key** bleibt auf dem Computer des Benutzers
3. Der **Public Key** wird an den Server übergeben
4. Der Server speichert den **Public Key**
5. Beim Login überprüft der Server, ob der Client den passenden **Private Key** besitzt
6. Wenn beide Schlüssel zusammenpassen, wird der Zugriff erlaubt

---

## 2. Aufbau eines SSH-Schlüsselpaars

Ein SSH-Schlüsselpaar besteht aus zwei Dateien:

| Schlüssel | Beschreibung |
|----------|--------------|
| Privater Schlüssel | bleibt auf dem Computer des Benutzers |
| Öffentlicher Schlüssel | wird auf dem Server gespeichert |

Wichtige Regel:

- Der **private Schlüssel darf niemals weitergegeben werden**
- Nur der **öffentliche Schlüssel** wird an den Server oder den Administrator übermittelt

---

## 3. SSH-Schlüssel auf einem Windows-Client erstellen

### Schritt 1 – PowerShell öffnen

1. Startmenü öffnen  
2. **PowerShell** suchen  
3. PowerShell starten  

### Schritt 2 – SSH-Schlüssel generieren

Folgenden Befehl ausführen:

```powershell
ssh-keygen -t ed25519 -C "projekt-ssh"
```

Danach stellt das System einige Fragen:

```text
Enter file in which to save the key
```

→ **Enter drücken**, um den Standardpfad zu verwenden

```text
Enter passphrase
```

→ optional, aber empfohlen

### Standardpfad unter Windows

Der SSH-Schlüssel wird standardmäßig hier gespeichert:

```text
C:\Users\BENUTZERNAME\.ssh\
```

Es entstehen zwei Dateien:

| Datei | Bedeutung |
|------|-----------|
| id_ed25519 | Privater Schlüssel |
| id_ed25519.pub | Öffentlicher Schlüssel |

---

## 4. Öffentlichen Schlüssel auf dem Windows-Client anzeigen

Um den öffentlichen Schlüssel anzuzeigen, wird in PowerShell folgender Befehl verwendet:

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

Beispielausgabe:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... user@pc
```

Die **gesamte Zeile** wird kopiert und an den Administrator übermittelt.

---

## 5. SSH-Zugriff auf Linux-Server

### 5.1 Public Key manuell auf einem Linux-Server hinterlegen

Wenn der Public Key direkt auf einem Linux-Server eingetragen werden soll:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

Den erhaltenen Public Key als neue Zeile einfügen, speichern und anschließend die Rechte korrekt setzen:

```bash
chmod 600 ~/.ssh/authorized_keys
```

Diese Methode funktioniert auf jedem Linux-Server.

---

### 5.2 Public Key über Google Cloud Console hinzufügen

Der Public Key kann direkt über die Google Cloud Console hinterlegt werden.

Pfad:

```text
Google Cloud Console
→ Compute Engine
→ VM Instances
→ gewünschte VM auswählen
→ Edit
→ SSH Keys
```

Dort wird der Public Key eingefügt und gespeichert.

Diese Methode ist besonders praktisch, wenn der Linux-Server in Google Cloud betrieben wird.

---

### 5.3 SSH-Verbindung von einem Windows-Client zu einem Linux-Server herstellen

Nach erfolgreicher Hinterlegung des Public Keys kann sich der Benutzer mit dem Server verbinden.

Format:

```powershell
ssh BENUTZERNAME@SERVER_IP
```

Beispiel:

```powershell
ssh user@34.51.139.200
```

Falls der Private Key nicht im Standardpfad liegt, kann er explizit angegeben werden:

```powershell
ssh -i $env:USERPROFILE\.ssh\id_ed25519 BENUTZERNAME@SERVER_IP
```

---

### 5.4 Verbindung prüfen (Linux-Server)

Nach erfolgreicher Verbindung erscheint eine Shell des Servers:

```text
benutzername@server:~$
```

Prüfen:

```bash
whoami
```

---

### 5.5 Fehlerbehebung (Linux-Server)

Bei Problemen können die Berechtigungen geprüft werden:

```bash
ls -ld ~/.ssh
ls -l ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

## 6. SSH-Zugriff auf Windows-Server

Damit ein Windows-Server SSH-Key-Authentifizierung unterstützt, muss **OpenSSH Server** installiert und aktiviert sein.

### 6.1 OpenSSH Server auf Windows installieren

PowerShell als Administrator öffnen:

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

### 6.2 OpenSSH-Dienst starten und aktivieren

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

Status prüfen:

```powershell
Get-Service sshd
```

---

### 6.3 Firewall-Regel für SSH prüfen oder hinzufügen

```powershell
Get-NetFirewallRule -Name *OpenSSH*
```

Falls nötig, Regel hinzufügen:

```powershell
New-NetFirewallRule -Name sshd `
  -DisplayName "OpenSSH Server (sshd)" `
  -Enabled True `
  -Direction Inbound `
  -Protocol TCP `
  -Action Allow `
  -LocalPort 22
```

---

### 6.4 Public Key auf einem Windows-Server hinterlegen

Es gibt zwei typische Varianten.

#### Variante A – Benutzerbezogen

Der Public Key wird in der Datei `authorized_keys` des Benutzers gespeichert.

Standardpfad:

```text
C:\Users\BENUTZERNAME\.ssh\authorized_keys
```

PowerShell-Befehle:

```powershell
New-Item -ItemType Directory -Force -Path $env:USERPROFILE\.ssh
notepad $env:USERPROFILE\.ssh\authorized_keys
```

Den Public Key in die Datei einfügen und speichern.

Inhalt prüfen:

```powershell
Get-Content $env:USERPROFILE\.ssh\authorized_keys
```

#### Variante B – Administrator-Konto auf Windows-Server

Für administrative Konten wird häufig folgende Datei verwendet:

```text
C:\ProgramData\ssh\administrators_authorized_keys
```

PowerShell-Befehle:

```powershell
notepad C:\ProgramData\ssh\administrators_authorized_keys
Get-Content C:\ProgramData\ssh\administrators_authorized_keys
```

Den Public Key einfügen und speichern.

---

### 6.5 Berechtigungen auf Windows-Servern setzen

Damit OpenSSH die Key-Dateien akzeptiert, müssen die Berechtigungen korrekt gesetzt sein.

#### Rechte für `administrators_authorized_keys`

```powershell
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "Administrators:F"
icacls C:\ProgramData\ssh\administrators_authorized_keys /grant "SYSTEM:F"
```

#### Rechte für benutzerbezogene `authorized_keys`

```powershell
icacls $env:USERPROFILE\.ssh /inheritance:r
icacls $env:USERPROFILE\.ssh /grant "$env:USERNAME:F"
icacls $env:USERPROFILE\.ssh\authorized_keys /inheritance:r
icacls $env:USERPROFILE\.ssh\authorized_keys /grant "$env:USERNAME:F"
```

---

### 6.6 SSH-Verbindung von einem Windows-Client zu einem Windows-Server herstellen

Format:

```powershell
ssh BENUTZERNAME@SERVER_IP
```

Beispiel:

```powershell
ssh Administrator@192.168.1.50
```

Oder mit explizitem Private Key:

```powershell
ssh -i $env:USERPROFILE\.ssh\id_ed25519 Administrator@192.168.1.50
```

---

### 6.7 Verbindung prüfen (Windows-Server)

Nach erfolgreicher Verbindung können folgende Befehle verwendet werden:

```powershell
whoami
hostname
```

---

### 6.8 Fehlerbehebung (Windows-Server)

Wichtige Prüfungen:

```powershell
Get-Service sshd
Get-Content C:\ProgramData\ssh\administrators_authorized_keys
Get-Content $env:USERPROFILE\.ssh\authorized_keys
Restart-Service sshd
```

---

## 7. Sicherheitshinweise

Für den sicheren Umgang mit SSH-Schlüsseln gelten folgende Regeln:

- Jeder Benutzer sollte **sein eigenes SSH-Schlüsselpaar** besitzen
- Der **private Schlüssel** bleibt immer lokal auf dem Client
- Nur der **öffentliche Schlüssel** darf weitergegeben werden
- SSH-Zugriffe sollten nur autorisierten Benutzern erlaubt werden
- Passwort-Login sollte möglichst vermieden oder deaktiviert werden

---

## 8. Zusammenfassung

Der SSH-Zugriff mit Schlüsseln funktioniert nach einem einfachen und sicheren Prinzip:

1. Der Client erstellt ein SSH-Schlüsselpaar  
2. Der Public Key wird an den Server übergeben  
3. Der Server speichert den Public Key  
4. Der Benutzer verbindet sich mit seinem Private Key per SSH  

Je nach Servertyp kann der Public Key auf unterschiedliche Weise hinterlegt werden:

- direkt auf einem **Linux-Server** in `authorized_keys`
- über die **Google Cloud Console** im Bereich **SSH Keys**
- auf einem **Windows-Server** in `authorized_keys` oder `administrators_authorized_keys`
