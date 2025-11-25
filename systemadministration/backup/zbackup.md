# zbackup

**more secure to use an editor**

```sh
mkdir  /opt/backup/zbackup

echo mypassword > ~/.my_backup_password
chmod 600 ~/.my_backup_password

zbackup init --password-file ~/.my_backup_password /opt/backup/zbackup

```

```sh
tar c /opt | zbackup --password-file ~/.my_backup_password backup /opt/backup/zbackup/repo/backups/backup-`date '+%Y-%m-%d'`
zbackup --password-file ~/.my_backup_password restore /opt/backup/zbackup/repo/backups/backup-`date '+%Y-%m-%d'` > /opt/backup-restored.tar
```

