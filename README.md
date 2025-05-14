# Tron - HackMyVM (Easy)
 
![Tron.png](Tron.png)

## Übersicht

*   **VM:** Tron
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Tron)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** (Datum konnte dem Bericht nicht entnommen werden)
*   **Original-Writeup:** https://alientec1908.github.io/Tron_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel der "Tron"-Challenge war die Erlangung von User- und Root-Rechten. Der Weg begann mit der Enumeration eines Webservers (Port 80), der einen Hinweis (`/robots.txt` -> `/??????`) auf eine Seite mit einem HTML-Kommentar enthielt: ``. Ein weiterer Fund im Verzeichnis `/MCP/tron.txt` war ein Base64-kodierter String, der zu Brainfuck-Code dekodierte. Die Ausführung des Brainfuck-Codes ergab den Benutzernamen `player`. In `/MCP/terminalserver/30513.txt` wurde ein Hinweis auf einen Atbash-Cipher gefunden. Die Anwendung dieses Ciphers auf den Passwort-Teil aus dem HTML-Kommentar (`SbWP9q94ZtE9qD`) ergab das Passwort `HyDK9j94AgV9jW`. Dies ermöglichte den SSH-Login als `player:HyDK9j94AgV9jW`. Die User-Flag wurde in dessen Home-Verzeichnis gefunden. Die Privilegieneskalation zu Root erfolgte durch die Entdeckung eines Benutzers `hacker` in `/etc/passwd` mit UID/GID 0. Nachdem dessen Passwort (im Log nicht explizit gezeigt, wie es gefunden wurde) erlangt wurde, konnte mittels `su hacker` Root-Zugriff erlangt werden.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   Web Browser (View Source)
*   `base64` (Decoder)
*   Brainfuck Interpreter (z.B. dcode.fr)
*   Substitution Cipher (Atbash) Decoder (z.B. dcode.fr)
*   `ssh`
*   `sudo` (versucht)
*   `su`
*   `cat`
*   `nano` (oder anderer Texteditor, impliziert)
*   `openssl passwd` (versucht)
*   Standard Linux-Befehle (`ls`, `cd`, `id`, `pwd`, `export`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Tron" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration (Clue Hunting):**
    *   IP-Findung mit `arp-scan` (`192.168.2.115`).
    *   `nmap`-Scan identifizierte offene Ports: 22 (SSH - OpenSSH 7.9p1) und 80 (HTTP - Apache 2.4.38).
    *   `gobuster` auf Port 80 fand `/robots.txt`, `/MCP/` und andere Standardpfade.
    *   `/robots.txt` enthielt einen Hinweis auf `/??????`.
    *   Der Quellcode von `http://192.168.2.115/??????` enthielt den Kommentar `` (Benutzername `kzhh`, Passwort-Rohdaten `SbWP9q94ZtE9qD`).
    *   In `http://192.168.2.115/MCP/tron.txt` wurde ein Base64-String gefunden, der zu Brainfuck-Code dekodierte (`++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>++++++++++++.----.-----------.++++++++++++++++++++++++.--------------------.+++++++++++++.`).
    *   Die Ausführung des Brainfuck-Codes ergab den Benutzernamen `player`.
    *   In `http://192.168.2.115/MCP/terminalserver/30513.txt` wurde ein Hinweis auf einen Atbash-Cipher gefunden.
    *   Anwendung des Atbash-Ciphers auf die Passwort-Rohdaten `SbWP9q94ZtE9qD` ergab das Passwort `HyDK9j94AgV9jW`.

2.  **Initial Access (SSH als `player`):**
    *   Erfolgreicher SSH-Login als `player` mit dem Passwort `HyDK9j94AgV9jW` (`ssh player@tron.vm`).
    *   User-Flag `HMVMuserplayer2021` in `/home/player/user.txt` gelesen.

3.  **Privilege Escalation (von `player` zu `root` via `hacker`-Account):**
    *   `sudo -l` für `player` zeigte keine Sudo-Rechte.
    *   Analyse von `/etc/passwd` offenbarte einen Benutzer `hacker` mit UID 0 und GID 0 (`hacker:$6$...:0:0:root:/root:/bin/bash`).
    *   *Der genaue Weg zur Erlangung des Passworts für den `hacker`-Account ist im Log nicht dokumentiert.*
    *   Wechsel zum Benutzer `hacker` (effektiv `root`) mittels `su hacker` und dem (nicht gezeigten) Passwort.
    *   Erlangung einer Root-Shell.
    *   Root-Flag `HMVMMasterControlProgram2021` in `/root/root.txt` gelesen.

## Wichtige Schwachstellen und Konzepte

*   **Hinweise in `robots.txt`:** Führte zu einer Seite mit weiteren Hinweisen.
*   **Credentials/Hinweise in HTML-Kommentaren:** Enthielt einen Teil der Zugangsdaten.
*   **Steganographie/Verschleierung durch mehrere Chiffren:**
    *   Base64-kodierter Brainfuck-Code in einer Textdatei zur Enthüllung des Benutzernamens.
    *   Atbash-Cipher (dessen Existenz in einer anderen Datei offenbart wurde) zur Dekodierung des Passworts.
*   **Fehlkonfigurierter Benutzeraccount (`hacker` mit UID 0):** Ein Nicht-Root-Benutzer besaß Root-Privilegien, was eine direkte Eskalation nach Passwortkompromittierung ermöglichte.

## Flags

*   **User Flag (`/home/player/user.txt`):** `HMVMuserplayer2021`
*   **Root Flag (`/root/root.txt`):** `HMVMMasterControlProgram2021`

## Tags

`HackMyVM`, `Tron`, `Easy`, `Steganography`, `Base64`, `Brainfuck`, `Atbash Cipher`, `SSH`, `Misconfiguration`, `/etc/passwd`, `Privilege Escalation`, `Linux`, `Web`
