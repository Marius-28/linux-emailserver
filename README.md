# linux-emailserver
How i install an Managed Email Server only with linux


Um einen E-Mail-Server auf Ubuntu zu installieren, der ausschließlich Open-Source-Software verwendet und die von dir gewünschten Funktionen wie Administration mit ISPConfig, automatische Backups, sowie Spam- und Virenfilterung umfasst, kann folgende Open-Source-Software verwendet werden:

### 1. **E-Mail-Server Software**
   Für einen kompletten E-Mail-Server kannst du **Postfix** (für den Versand von E-Mails), **Dovecot** (für IMAP und POP3) und **SpamAssassin** sowie **ClamAV** (für die Viren- und Spamfilterung) verwenden.

   - **Postfix**: Ein beliebter MTA (Mail Transfer Agent) für den Versand von E-Mails.
   - **Dovecot**: Ein MDA (Mail Delivery Agent) für den Empfang von E-Mails.
   - **SpamAssassin**: Ein Open-Source-Spamfilter, der Spam-Mails identifiziert und blockiert.
   - **ClamAV**: Ein Open-Source-Virenscanner für die E-Mail-Filterung.

### 2. **ISPConfig für die Verwaltung**
   ISPConfig ist eine Web-basierte Verwaltungsoberfläche, mit der du den E-Mail-Server verwalten kannst. Es ermöglicht das Verwalten von Domains, E-Mail-Konten, Spamfilter, sowie das Einrichten von Backups und weiteren Funktionen.

### 3. **Automatische Backups und Rotation**
   Für die automatische Backup-Erstellung und -Rotation kannst du ein Skript und `cron` verwenden, um regelmäßig Backups zu erstellen und alte Backups zu löschen. Eine gute Wahl wäre auch, **rsync** und **tar** zu nutzen.

### 4. **Viren- und Spamfilterung**
   Wie bereits erwähnt, verwenden wir **ClamAV** für Virenscans und **SpamAssassin** für die Spamfilterung. Beide können in Postfix und Dovecot integriert werden.

---

### Schritt-für-Schritt-Anleitung zur Installation

#### 1. **Ubuntu Server vorbereiten**
   Beginne mit der Installation eines frischen Ubuntu Servers. Aktualisiere zunächst das System:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

#### 2. **Postfix und Dovecot installieren**
   Installiere Postfix, Dovecot und einige zusätzliche Pakete für die Integration:

   ```bash
   sudo apt install postfix dovecot-core dovecot-imapd dovecot-lmtpd mailutils -y
   ```

   Wähle bei der Postfix-Installation „Internet Site“ und gib den Hostnamen deines Servers ein.

#### 3. **SpamAssassin und ClamAV installieren**
   Installiere SpamAssassin und ClamAV, um die E-Mails zu filtern:

   ```bash
   sudo apt install spamassassin clamav clamav-daemon -y
   ```

   Stelle sicher, dass der ClamAV-Daemon startet:

   ```bash
   sudo systemctl enable clamav-daemon
   sudo systemctl start clamav-daemon
   ```

   Konfiguriere SpamAssassin, um es mit Postfix zu integrieren. In der Datei `/etc/spamassassin/local.cf` kannst du Parameter wie `required_score` und `report_safe` anpassen.

#### 4. **ISPConfig installieren**
   ISPConfig kann direkt aus den offiziellen Quellen installiert werden. Befolge dazu die Installationsanweisungen auf der [ISPConfig-Website](https://www.ispconfig.org/), um sicherzustellen, dass du die neueste Version installierst.

   Zum Beispiel:

   ```bash
   wget -O - https://get.ispconfig.org | sh
   ```

   Folge den Anweisungen des Installationsscripts. ISPConfig kann auch direkt die E-Mail-Dienste konfigurieren und verwalten, wenn es einmal installiert ist.

#### 5. **Konfiguration des Postfix Servers**
   - Bearbeite die Konfigurationsdatei von Postfix, um sicherzustellen, dass SpamAssassin und ClamAV korrekt integriert sind:

     ```bash
     sudo nano /etc/postfix/main.cf
     ```

   - Füge folgende Zeilen hinzu, um den Spamfilter und die Virenprüfung zu aktivieren:

     ```bash
     content_filter = smtp-amavis:[127.0.0.1]:10024
     ```

   - Konfiguriere auch das DKIM/DMARC/SPF für bessere E-Mail-Sicherheit (optional).

#### 6. **Automatische Backups und Rotation einrichten**
   Erstelle ein Skript für die Sicherung deiner E-Mail-Daten. Zum Beispiel:

   ```bash
   sudo nano /etc/cron.daily/email_backup.sh
   ```

   Beispiel-Skript für Backups:

   ```bash
   #!/bin/bash
   BACKUP_DIR="/backup/emails"
   DATE=$(date +%Y-%m-%d)
   mkdir -p $BACKUP_DIR/$DATE
   rsync -a /var/mail/ $BACKUP_DIR/$DATE
   find $BACKUP_DIR/* -mtime +30 -exec rm -rf {} \;  # Löscht Backups, die älter als 30 Tage sind
   ```

   Mache das Skript ausführbar:

   ```bash
   sudo chmod +x /etc/cron.daily/email_backup.sh
   ```

#### 7. **Email-Filterung mit SpamAssassin und ClamAV**
   - **SpamAssassin**: Konfiguriere SpamAssassin, um es mit Postfix und Dovecot zu verwenden. Dazu kannst du `/etc/postfix/main.cf` und `/etc/spamassassin/local.cf` anpassen.
   - **ClamAV**: Integriere ClamAV mit Postfix, indem du `amavisd-new` als Content-Filter verwendest.

     Installiere `amavisd-new`:

     ```bash
     sudo apt install amavisd-new -y
     ```

     Bearbeite die Konfiguration von `amavisd-new` und aktiviere die Verwendung von ClamAV und SpamAssassin.

#### 8. **Testen und Verwalten**
   - Teste die Konfiguration mit `telnet` oder einem Mail-Client.
   - Verwende ISPConfig, um E-Mail-Konten, Spam-Filter und Backups zu verwalten.

---

### Fazit
Diese Anleitung zeigt die grundlegende Einrichtung eines Open-Source-E-Mail-Servers mit Ubuntu, Postfix, Dovecot, SpamAssassin, ClamAV und ISPConfig. Damit hast du einen voll funktionsfähigen, sicheren und skalierbaren E-Mail-Server mit Spam- und Virenschutz sowie einer praktischen Verwaltung über ISPConfig.
