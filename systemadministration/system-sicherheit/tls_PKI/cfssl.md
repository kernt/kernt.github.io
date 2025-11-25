# cfssl


_csr.json für local domain_

```json
{
    "hosts": [
        "srv2.local",
        "https://srv2.local",
        "tobkern1980@gmail.com",
        "192.168.4.56"
    ],
    "key": {
        "algo": "rsa",
        "size": 4096
    },
    "names": [
        {
            "C":  "DE",
            "L":  "Hamburg",
            "O":  "IT",
            "OU": "IT",
            "ST": "Hamburg"
        }
    ]
}
```

_csr.json für tuxsupport.de domain_

```json

```



_cfssl_sign.sh_

```sh
#!/bin/bash
#
cfssl sign -ca     /etc/ssl/certs/cfssl_local.pem      \
           -ca-key /etc/ssl/private/cfssl_local_key.pem \
           -hostname srv2.local               \
           ./local.pem
```


_generate ca_

```sh

```

_Zertifikat request und private key erzeugen_

`cfssl genkey csr.json`

_Generate self-signed root CA certificate und private key_

`cfssl genkey -initca csr.json | cfssljson -bare ca`

_Generriere einen local-issued zertifikate und private key_

`cfssl gencert -ca ./ca.pem -ca-key ./ca-key.pem -hostname=srv2 csr.json`

[github cfssl](https://github.com/cloudflare/cfssl)