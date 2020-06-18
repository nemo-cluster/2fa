# 2FA
Erste Tests mit 2FA bei NEMO zum Testen für bwHPC

Bei diesem Ansatz wird nur ein SSH-Zugang des Nutzers benötigt und der QR-Code für Auth-Apps wird angezeigt. Dieser wird nur beim ersten Login angezeigt.

Zukünftig könnte man auch Yubikeys etc. unterstützen. Derzeit funktioniert das nur nur über die Yubikey-Mobil-App, die vergleichbar mit den Auth-Apps ist.

## Mobile-Apps

Mobile Authenticator-Apps für Andriod

* [FreeOTP Authenticator](https://play.google.com/store/apps/details?id=org.fedorahosted.freeotp)
* [Aegis Authenticator ](https://play.google.com/store/apps/details?id=com.beemdevelopment.aegis)
* [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2)
* [Yubico Authenticator](https://play.google.com/store/apps/details?id=com.yubico.yubioath)

und IOS

* [FreeOTP Authenticator](https://apps.apple.com/us/app/freeotp-authenticator/id872559395)
* [Google Authenticator](https://apps.apple.com/us/app/google-authenticator/id388497605)
* [Yubico Authenticator](https://apps.apple.com/us/app/yubico-authenticator/id1476679808)

## Vorbereitung

Es gibt drei Knotenarten, den Knoten auf dem der "Zweite Faktor" erzeugt wird, einen zentralen Managementknoten und als letztes, die Knoten auf denen die Zwei-Faktor-Authentifizierung erfolgen soll.

### QR-Code-Generator-Knoten

Der QR-Code-Generator benötigt Software zu Erzeugen des "Zweiten Faktors".

Installation von Paketabhängigkeiten bei CentOS7
```bash
# EPEL Repo
yum install -y epel-release
# oathtool, QR-Code-Generator und inotify-tools
yum install -y oathtool qrencode inotify-tools pinentry
```

Wenn Nutzer-Homes automatisch angelegt werden sollen, Oddjob installieren
```bash
yum install -y oddjob-mkhomedir
authconfig --enablemkhomedir --update
```

Weitere sinnvolle Softwarepakete:
* SSSD für die Nutzerauthentifikation
* Firewalld für zusätzliche Firewalls gegen Angreifer
* OpenSSL, sollte bereits installiert sein

### Management-Knoten

Der Management-Knoten benötigt neben `<openssl>` zum Entschlüsseln und Erstellen der Schlüssel, `<scp>` oder `<rsync>` und beispielsweise `<pdsh>` zum Kopieren der Dateien auf die Knoten.

```bash
# EPEL Repo
yum install -y epel-release
# pdsh
yum install -y pdsh pdsh-mod-genders
```

### Zwei-Faktor-Knoten

Damit der "Zweite Faktor" geprüfte werden kann wird nur folgendes Paket benötigt.
```bash
# EPEL Repo
yum install -y epel-release
# pam_oath
yum install -y pam_oath
```

## OATH-Skripte zum Generieren von Codes einrichten

In der neuen Version werden die Schlüssel für den "Zweiten Faktor" von den Nutzern erzeugt und direkt mit einem öffentlichen Schlüssel des Management-Knotens verschlüsselt. Ein Systemd-Dienst verschiebt die Datei in ein zentralen Ordner mit Hilfe von `<inotifywait>`.

### Management-Knoten

OATH-Skript kopieren
```bash
cp oathcron /usr/local/bin/
chmod 755 oathcron
```

Erstellen der Schlüssel zum verschlüsseln der "Zweiten Faktoren"
```bash
gpg2 --gen-key
gpg --export key@subission.binac > /etc/public_key.gpg
```

Crontab-Eintrag einrichten, sollte erst ganz zum Schluss eingerichtet werden.
```bash
$ crontab -e
* * * * *       /usr/local/bin/oathcron
```

### QR-Code-Genarator-Knoten

OATH-Skripte kopieren
```bash
cp oathinotify oathgen /usr/local/bin/
cp 2fa-inotify.service /etc/systemd/system/
chmod 755 oathgen oathinotify
```

Systemd-Service aktivieren
```bash
systemctl daemon-reload
systemctl enable 2fa-inotify.service
systemctl start 2fa-inotify.service
```

Damit die Nutzer*innen nur ein Skript ausführen können, wird nur das in des SSHD-Konfiguration gestattet.

SSHD-Konfiguration `/etc/ssh/sshd_config` anpassen
```bash
# alle außer root führen automatisch Kommando aus
Match User *,!root
        ForceCommand /usr/local/bin/oathgen
```

 SSH-Dienst neu laden
 ```bash
 systemctl reload sshd.service
```

### Zwei-Faktor-Knoten

Installation von Paketabhängigkeiten bei CentOS7
```bash
# EPEL Repo
yum install -y epel-release
# PAM OATH
yum install -y pam_oath
```

PAM konfigurieren für SSH `/etc/pam.d/sshd`. Je nachdem ob diese Zeile am Anfang oder Ende steht wird das TOTP-Token vor oder nach Passworteingabe abgefragt.
```bash
auth	  required pam_oath.so usersfile=/usr/local/etc/users.oath window=30 digits=6
```

SSHD für Passwort und TOTP konfigurieren `/etc/ssh/sshd_config`
```bash
ChallengeResponseAuthentication yes
PasswordAuthentication no
UsePAM yes
```

Sollen Zusätzlich SSH-Schlüssel mit TOTP abgesichert werden, dann muss folgende Zeile in der SSHD-Konfiguration enthalten sein:
```bash
PasswordAuthentication no
ChallengeResponseAuthentication yes
PubkeyAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
UsePAM yes
```
