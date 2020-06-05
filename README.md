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

Installation von Paketabhängigkeiten bei CentOS7
```bash
# EPEL Repo
yum install -y epel-release
# OATH inkl. zusätzlicher Werkzeuge
yum install -y oathtool pam_oath qrencode
```

Wenn Nutzer-Homes automatisch angelegt werden sollen, Oddjob installieren
```bash
yum install -y oddjob-mkhomedir
authconfig --enablemkhomedir --update
```

## OATH-Skripte zum Generieren von Codes einrichten

Es gibt zwei Versionen, für die eine werden Sudo-Rechte benötigt, die andere erledigt die Root-Aufgaben über einen Cron-Job.

### Variante 1 - Cron-Job

OATH-Skripte kopieren
```bash
cp oathcron oathgen /usr/local/bin/
chmod 755 oathgen
chmod 700 oathcron
```

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

Crontab anlegen `crontab -e`
```bash
* * * * *      /usr/local/bin/oathcron
```

### Variante 2 - Sudo

OATH-Skripte kopieren
```bash
cp oathuseradd oathgenerate /usr/local/bin/
chmod 755 oathgenerate
chmod 700 oathuseradd
```

SSHD-Konfiguration `/etc/ssh/sshd_config` anpassen
```bash
# alle außer root führen automatisch Kommando aus
Match User *,!root
        ForceCommand /usr/local/bin/oathgenerate
```

 SSH-Dienst neu laden
 ```bash
 systemctl reload sshd.service
```

Sudo anlegen `/etc/sudoers.d/10-oathuseradd`
```bash
ALL ALL=(root) NOPASSWD: /usr/local/bin/oathuseradd
```

## Konfiguration der Login-Knoten

Installation von Paketabhängigkeiten bei CentOS7
```bash
# EPEL Repo
yum install -y epel-release
# PAM OATH
yum install -y pam_oath
```

PAM konfigurieren für SSH `/etc/pam.d/sshd`. Je nachdem ob diese Zeile am Anfang oder Ende steht wird das TOTP-Token vor oder nach Passworteingabe abgefragt.
```bash
auth	  required pam_oath.so usersfile=/etc/users.oath window=30 digits=6
```

SSHD für Passwort und TOTP konfigurieren `/etc/ssh/sshd_config`
```bash
ChallengeResponseAuthentication yes
UsePAM yes
```

Sollen Zusätzlich SSH-Schlüssel mit TOTP abgesichert werden, dann muss folgende Zeile in der SSHD-Konfiguration enthalten sein:
```bash
AuthenticationMethods publickey,keyboard-interactive
```

## Pro und Contra beider Versionen

TODO
