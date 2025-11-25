`sudo systemctl status psacct`

You see the status showing as disabled, so let’s start it manually using the following commands, which will create a **/var/account/pacct** file.

```sh
sudo systemctl start psacct
sudo systemctl enable psacct
sudo systemctl status psacct
```

