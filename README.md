# Driftingblues3 - HackMyVM (Easy)

![Driftingblues3.png](Driftingblues3.png)

## Übersicht

*   **VM:** Driftingblues3
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Driftingblues3)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 22. November 2022
*   **Original-Writeup:** https://alientec1908.github.io/Driftingblues3_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser "Easy"-Challenge war es, Root-Zugriff auf der Maschine "Driftingblues3" zu erlangen. Die Enumeration eines Webservers (Apache) enthüllte den Pfad `/eventadmins`, der Hinweise auf einen Benutzer `john` und ein "poisonous" SSH-Problem enthielt. Eine weitere Datei (`/littlequeenofspades.html`), referenziert aus `/eventadmins`, enthielt Base64-kodierte Hinweise auf den Pfad `/adminsfixit.php`. Diese Seite zeigte SSH-Authentifizierungslogs. Durch einen SSH-Loginversuch mit einer PHP-Payload (`ssh ''@host`) als Benutzername wurde eine RCE-Schwachstelle getriggert. Befehle konnten dann über den `cmd`-Parameter von `/adminsfixit.php` ausgeführt werden, was zu einer initialen Shell als `www-data` führte. Als `www-data` wurde ein SUID-Root-Binary `/usr/bin/getinfo` gefunden. Die Ausführung dieses Binaries ermöglichte die Eskalation zu Root.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `hydra` (Versuch, erfolglos wegen Key-Auth)
*   `curl` (oder Browser)
*   `base64`
*   `nc` (netcat)
*   `python3` (für pty-Shell-Stabilisierung)
*   `find`
*   `sudo` (versucht, nicht verfügbar)
*   `/usr/bin/getinfo` (SUID-Binary als Exploit-Vektor)
*   Standard Linux-Befehle (`ssh`, `cat`, `echo`, `export`, `stty`, `ls`, `id`, `cd`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Driftingblues3" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.120`, Hostname `driftingblues`).
    *   `nmap`-Scan identifizierte SSH (22/tcp) und Apache (80/tcp). `robots.txt` verwies auf `/eventadmins`.
    *   `gobuster` fand diverse Verzeichnisse, aber nicht unmittelbar `/eventadmins`.
    *   Die manuelle Untersuchung von `/eventadmins` enthielt Hinweise auf ein SSH-Problem, den Benutzer `john` ("it's poisonous!!!") und die Datei `/littlequeenofspades.html`.
    *   `/littlequeenofspades.html` enthielt einen Base64-kodierten String, der zu `/adminsfixit.php` dekodiert wurde.
    *   `/adminsfixit.php` zeigte SSH-Authentifizierungslogs.

2.  **Initial Access (als `www-data` via RCE):**
    *   Ein SSH-Login-Versuch mit einer PHP-Payload als Benutzername (`ssh ''@192.168.2.120`) wurde durchgeführt. Obwohl der Login fehlschlug, wurde die Payload im System verarbeitet.
    *   Anschließend konnte über die URL `http://192.168.2.120/adminsfixit.php?cmd=[BEFEHL]` Remote Code Execution erreicht werden, da die Ausgabe des Befehls in die SSH-Logs (und somit auf die Webseite) geschrieben wurde.
    *   Mittels dieser RCE-Schwachstelle wurde eine Bash-Reverse-Shell zu einem Netcat-Listener des Angreifers gestartet.
    *   Erfolgreicher Shell-Zugriff als `www-data`.

3.  **Privilege Escalation (von `www-data` zu `root` via SUID Binary):**
    *   Als `www-data` wurde die Suche nach SUID-Dateien durchgeführt (`find / -type f -perm -4000 ...`).
    *   Dabei wurde das SUID-Root-Binary `/usr/bin/getinfo` gefunden.
    *   Die Ausführung von `/usr/bin/getinfo` als `www-data` lieferte eine Root-Shell.
    *   Die User- und Root-Flags wurden gelesen.
    *(Ein inkonsistenter Block im Log bezüglich eines Benutzers `robertj` wurde ignoriert, da er nicht zum erfolgreichen Pfad passte.)*

## Wichtige Schwachstellen und Konzepte

*   **Information Disclosure:** Hinweise in HTML-Kommentaren, `robots.txt` und speziell präparierten Webseiten.
*   **Command/Code Injection via SSH Username:** Ein ungewöhnlicher Vektor, bei dem ein als SSH-Benutzername übergebener String serverseitig unsicher verarbeitet wurde, was RCE über eine andere Webseite ermöglichte.
*   **SUID Binary Exploit:** Ein benutzerdefiniertes SUID-Root-Binary (`/usr/bin/getinfo`) ermöglichte die direkte Eskalation zu Root.
*   **Base64 Encoding/Decoding:** Verwendet zur Verschleierung von Pfaden.

## Flags

*   **User Flag (Pfad nicht explizit im Log für den Fund, aber Flag genannt):** `5355B03AF00225CFB210AE9CA8931E51`
*   **Root Flag (`/root/root.txt`):** `dfb7f604a22928afba370d819b35ec83`

## Tags

`HackMyVM`, `Driftingblues3`, `Easy`, `Web`, `Apache`, `Information Disclosure`, `RCE`, `SSH Username Injection`, `SUID Binary`, `Privilege Escalation`, `Linux`
