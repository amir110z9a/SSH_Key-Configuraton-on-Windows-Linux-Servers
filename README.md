# SSH-Zugriff mit SSH-Schlüsseln

Diese README beschreibt kurz, wie der SSH-Zugriff auf **Linux- und Windows-Server** mit **SSH-Schlüsseln** eingerichtet wird.

## Grundprinzip

Ein Benutzer erstellt ein **SSH-Schlüsselpaar**:

- **Private Key** → bleibt auf dem eigenen Computer
- **Public Key** → wird auf dem Server gespeichert

Nur wenn der Client den passenden Private Key besitzt, wird der Zugriff erlaubt.

![SSH-Authentifizierung](ssh-diagramm.png)