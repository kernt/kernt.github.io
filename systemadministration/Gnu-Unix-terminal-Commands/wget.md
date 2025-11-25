# wget

**Save a file with a different name**

`wget -O filename url`

oder

` wget --output-document=file filename url`

**Download a file in background**

`wget -b url`

oder

`wget --background url`

**Resume a previous download**

`wget -c url`

oder

`wget --continue url`

**Limit the download speed**

`wget --limit-rate=1m url`

**Use URLs from the file to download**

`wget -i file`

oder

`wget --input-file file`

**Create a mirror of the website**

`wget -m url`

oder

`wget --mirror url`

**Download from web server using user credentials**