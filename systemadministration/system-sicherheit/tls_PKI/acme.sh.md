[Do API get Cert](https://my.do.de/api/letsencrypt?token=3MJ8mShwze7zSnEnssfp&domain=_acme-challenge.tuxcoach.de&value=OVxwaDm7MgN1IRG0eSivJMlepO9CL4X8vKo6Tcns)

```sh
export DO_LETOKEN="3MJ8mShwze7zSnEnssfp"
acme.sh --issue --dns dns_doapi -d tuxcoach.de -d *.tuxcoach.de
```

```sh
[Tue Nov  5 11:43:21 CET 2024] No EAB credentials found for ZeroSSL, let's obtain them
[Tue Nov  5 11:43:22 CET 2024] Registering account: https://acme.zerossl.com/v2/DV90
[Tue Nov  5 11:43:23 CET 2024] Registered
[Tue Nov  5 11:43:23 CET 2024] ACCOUNT_THUMBPRINT='WhLGZTme3xY9P1gjPE-WWzVe5YFWckS_WVJ5TRT1VcQ'
```


