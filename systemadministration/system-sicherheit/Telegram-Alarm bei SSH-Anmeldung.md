Der Instant Messaging-Dienst **Telegram** ist inzwischen in aller Munde. Der Dienst bietet auch die Möglichkeit Bots zu betreiben, die nützlich sein können, z. B. für Laufzeitalarmierungen. So z. B. auch bei SSH-Anmeldungen auf einem Server.

Zunächst muss ein Telegram-Bot angelegt werden. Wie das geht, wird auf [telegram.org](https://core.telegram.org/bots#6-botfather) beschrieben.

Ist ein Bot eingerichtet, kann man anhand des Keys des Bots und der eigenen Telegram-User-ID schon von irgendeinem Computer aus sich Nachrichten vom Bot schicken lassen. Das bedarf keiner weiteren Autorisierung oder Authentifizierung. Lediglich der Key des Bots ist ausschlaggebend, weshalb er auch sicher aufbewahrt werden sollte.

Die eigene User-ID bekommt man heraus, indem man z. B. **/start** an **@getmyid_bot** sendet. Die Antwort sieht dann z. B. so aus:
  
![](https://techgoat.net/images/261.png)

# Nun zum eigentlichen Clou:

Man legt ein Skript an, z. B. **/root/ssh_bot.sh**, mit dem folgenden Inhalt:

```
#!/bin/bash
USERID="Ihre eigene Telegram-ID"
KEY="Der Key / Token vom Bot"
URL="https://api.telegram.org/bot$KEY/sendMessage"
DATE_EXEC="$(date "+%b %-d %H:%M:%S")"
LINE=$(grep -E 'sshd.+Accepted' /var/log/auth.log | tail -n1 | sed 's/ \+/ /g' | cut -d' ' -f1-3,7,9,11 --output-delimiter=';')
DATE_LOG=$(echo $LINE | cut -d';' -f1-3 --output-delimiter=' '| cut -d':' -f1-3)
METHOD=$(echo $LINE | cut -d';' -f4)
USR=$(echo  $LINE | cut -d';' -f5)
IP=$(echo $LINE |cut -d';' -f6)
if [ "$DATE_EXEC" = "$DATE_LOG" ]; then
        C=$(/usr/bin/gil $IP "LAN" "192.168.0.0/24")
        if [ $? -ne 0 ]; then
                TEXT="$DATE_EXEC: $USR logged in to $(hostname) with $METHOD from $IP ($C)"
                curl -s --max-time 10 -d "chat_id=$USERID&disable_web_page_preview=1&text=$TEXT" $URL > /dev/null
        fi
fi
exit 0
```

Das **exit 0** ist wichtig, weil ansonsten so etwas aufkommen wird:  
![](https://techgoat.net/images/259.png)

Die Bestimmung des Landes, aus dem die Anmeldung initiiert wurde, beruht auf dem Artikel [Dienste mit Geo-IP-Blocker einschränken](https://techgoat.net/index.php?id=176) oder genauer gesagt auf das dort enthaltene Skript, **/usr/bin/gil**.

Dieses Skript muss natürlich noch eingetragen werden, dass es ausgeführt wird, sobald sich jemand per SSH anmeldet. Das geschieht, indem man einen Aufruf in der Datei **/etc/pam.d/sshd** initiiert. Hierfür muss eine Zeile ans Ende der Datei angefügt werden:

```
session optional pam_exec.so /root/ssh_bot.sh
```

Jetzt wird man per Telegram bei erfolgreichen Anmeldungen außerhalb des LANs informiert:  
![](https://techgoat.net/images/260.png)

