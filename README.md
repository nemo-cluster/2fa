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

## Varianten

Es gibt Variante A, die noch mit `openssl` RSA verschlüsselt und daten per `rsync` am Management-Knoten kopiert und per `pdcp` die Dateien verteilt und Variante B mit GPG-Verschlüsselung und `socat` als Mechanismus zum Kopieren der Credentials.

Die älteren Varianten sind auch noch mit Tags versehen.
