# 2FA
Erste Tests mit 2FA bei NEMO zum Testen für bwHPC

Bei diesem Ansatz wird nur ein SSH-Zugang des Nutzers benötigt und der QRcode für Auth-Apps wird angezeigt. Dieser wird nur beim ersten Login angezeigt.

Zukünftig könnte man auch Yubikeys etc. unterstützen. Derzeit funktioniert das nur nur über die Yubikey-Mobil-App, die vergleichbar mit den Auth-Apps ist.

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

## OATH-Skripte zum Generien von Codes einrichten

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

 SSH-Deinst neu laden
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

 SSH-Deinst neu laden
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

PAM konfugurieren für SSH `/etc/pam.d/sshd`. Je nachdem ob diese Zeile am Anfang oder Ende staht wird das TOTP-Token vor oder nach Passworteingabe abgefragt.
```bash
auth	  required pam_oath.so usersfile=/etc/users.oath window=30 digits=6
```

SSHD für PAsswort und TOTP konfigurieren `/etc/ssh/sshd_config`
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
