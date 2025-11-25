# How to Set Up APT Package Repository on Debian/Ubuntu Server

Last Updated: December 19th, 2022 [Guoan Xiao (Admin)](https://www.linuxbabe.com/author/xiao-guoan)

[3 Comments](https://www.linuxbabe.com/linux-server/set-up-package-repository-debian-ubuntu-server#comments)
[Linux Server](https://www.linuxbabe.com/category/linux-server)

This tutorial is going to show you how to set up your own package repository on Debian/Ubuntu server.

If you are a long-time Linux user, I’m sure you have installed software packages by adding an upstream repository such as [Spotify](https://www.linuxbabe.com/desktop-linux/install-stable-version-spotify-music-player-ubuntu) or [Jelly media server](https://www.linuxbabe.com/ubuntu/install-jellyfin-media-server-ubuntu-20-04). Most of the time you just enter `sudo apt install package_name` to install a software package from the default Debian/Ubuntu repository, but sometimes a package is only available in the upstream repository, or the upstream repository has the newest version of the software, so you need to use it.

If you are a developer or package maintainer, it can be very useful to run your own package repository to distribute your software packages to other Linux users, so they can install the software and get updates easily, and I will show you how in this article.

**Note**: This tutorial is not showing how to set up a local APT mirroring repository that replicates packages from the Debian/Ubuntu repository. The repository we are going to set up works like a Personal Package Archive (PPA).
## Requirements

Step 1 ~ Step 4 should be done on a local computer, to keep your private key separate from the cloud server. The private key should not be stored on a public-facing server and never share with anyone. The local computer and the cloud server should run Debian or Ubuntu, or a derivative distro.

You should prepare a [cloud Linux server](https://www.linuxbabe.com/linux-server/how-to-create-a-linux-vps-server-on-kamatera) and a [domain name](https://www.linuxbabe.com/namecheap).

## Step 1: Generate GPG Key Pair

You will need to sign software packages with a GPG private key and distribute the GPG public key along with the software packages, so users can verify the integrity of packages. If you don’t yet have a GPG key pair, please follow the tutorial linked below to create one.

- [A Practical Guide to GPG Part 1: Generate Your Public/Private Key Pair](https://www.linuxbabe.com/security/a-practical-guide-to-gpg-part-1-generate-your-keypair)

## Step 2: Create Your .Deb Package

`.deb` is the standard format for software packages developed for Debian/Ubuntu distros. You can grab an existing `.deb` file for the purpose of this tutorial, if the package is distributed under an open-source license. I will show you the basic steps to create a `.deb` file in case you need to develop your own software.

I use Firefox as an example. Ubuntu doesn’t ship Firefox in .deb format anymore. It only provides the Snap package for Firefox. I use Snap package sometimes, like [the Let’s Encypt client certbot](https://www.linuxbabe.com/linux-server/fix-common-lets-encrypt-certbot-errors), because it’s a simple program. I invoke it to obtain a TLS certificate, then don’t have to worry about it anymore. The web browser is a piece of software that I need to use all the time, but the Snap version of Firefox doesn’t integrate well with the file system, so I prefer to use the tranditional Debian package format.

Under the hood, a `.deb` file is just an `ar` achive, similar to a `.tar` achive. The following instructions are used to package a Firefox binary tarball into a `.deb` file. If you want to do packaging using a source code tarball, the steps are a little different.

CD to your home directory.

cd ~

Download Firefox tarball from Mozilla’s web server.

wget https://download-installer.cdn.mozilla.net/pub/firefox/releases/107.0.1/linux-x86_64/en-US/firefox-107.0.1.tar.bz2

Extract the archive.

tar xvf firefox-107.0.1.tar.bz2

Now there’s a `firefox` directory. Under this directory we need to create the `opt/firefox-deb/` sub-directory, because we want users to install the package to the `/opt/firefox-deb/` directory on their computer.

mkdir -p ~/firefox/opt/firefox-deb

Then open your file manager and move all other files and sub-directoris to the `~/firefox/opt/firefox-deb/` directory.

Next, Make a `DEBIAN` directory.

mkdir ~/firefox/DEBIAN

Create a control file under the DEBIAN directory.

nano ~/firefox/DEBIAN/control

Add the following lines to this file. The `Package` parameter will determine the package name.

Source: firefox
Maintainer: Xiao Guoan <xiao@linuxbabe.com>
Section: misc
Priority: optional
Standards-Version: 3.9.2
Build-Depends: debhelper (>= 9)
Package: firefox-deb 
Architecture: amd64
Description: Firefox web browser without the Snaps
Version: 107

Save and close the file. After that, we need to create a `.desktop` file, so user can easily start Firefox. Run the following command to create the usr/share/applications/ sub-directory.

mkdir -p ~/firefox/usr/share/applications/

Create the `.desktop` file.

nano ~/firefox/usr/share/applications/firefox-deb.desktop

For your convenience, I upload [my firefox-deb.desktop file](https://www.linuxbabe.com/public-share/firefox-deb.desktop) to my web server. You can download and use it.

Then run the following command to build the .deb package.

dpkg-deb --build ~/firefox

Ubuntu uses `zstd` compression for .deb packages. If you build .deb package for Debian on an Ubuntu computer, then you should use other compression methods like gzip, which will work on both Debian and Ubuntu.

dpkg-deb -Zgzip --build ~/firefox

## Step 3: Sign the Package with Your GPG Private Key

Install the `dpkg-sig` tool.

sudo apt install dpkg-sig

Then run the following command to sign the `.deb` package. It will use your default GPG private key, and you will need to enter your key passphrase to unlock it.

dpkg-sig --sign builder firefox.deb

## Step 4: Create the APT Repository

Install the Debian package repository producer.

sudo apt install reprepro

Then create a base directory for the repository.

sudo mkdir -p /var/www/repository/

Change the owner to your username.

sudo chown username:username /var/www/repository/

Create a `conf` sub-directory.

mkdir -p /var/www/repository/conf/

Create a text file named `distributions`.

nano /var/www/repository/conf/distributions

Add the following lines in this file.

Origin: repo.linuxbabe.com
Label: apt repository
Codename: jammy
Architectures: amd64 
Components: main
Description: LinuxBabe package repository for Debian/Ubuntu
SignWith: 752E173A3F8B04F5
Pull: jammy

Where

- `Origin`: the hostname of your repository.
- `Label`: Give it a name
- `Codename`: What distro your repository supports. `jammy` is for Ubuntu 22.04. If you want to support multiple distros, simple copy the above snippet, paste it in the same file and change the codename.
- `Architectures`: could be `amd64`, `i386`, or `source`.
- `Components`: If you don’t have lots of packages in your repository, use `main` as the single Component.
- `Description`: Describe what this repository is for.
- `SignWith`: The repository should be signed with a GPG key. A Release.gpg file will be generated. You need to enter your GPG key ID here.

You can find your key ID with the following command. Replace `user-id` with your GPG email address.

gpg --list-sigs user-id

Sample output:

![gpg list signature](https://www.linuxbabe.com/wp-content/uploads/2016/02/gpg-list-signature.png)

As you can see, my key ID is _752E173A3F8B04F5._

Save and close the file. Next, add the .deb file to the repository. You will be asked to enter your GPG key passphrase.

reprepro -V --basedir /var/www/repository/ includedeb jammy /path/to/the/.deb_file

- `-V`: verbose mode.
- `--basedir`: specify the base directory.
- `includedeb`: add deb package to the repository.
- `jammy`: the distro codename. In this case, the deb package will be added for Ubuntu 22.04 users.

![reprepo create repository structure](https://www.linuxbabe.com/wp-content/uploads/2022/12/reprepo-create-repository-structure.png)

Note that you should not run reprepro as `root` or with `sudo`, or it can’t find your GPG key.

Now we should also add the GPG public key to the repository. Use the following command to export your public key and save it under the repository base directory. `user-id` is your email address of your GPG key.

gpg --armor --export user-id | sudo tee /var/www/repository/gpg-pubkey.asc

![APT repostiory public key](https://www.linuxbabe.com/wp-content/uploads/2022/12/APT-repostiory-public-key.png)

## Step 5: Upload the Repository to a Cloud Server

If you want others to use your repository, then you should create the repository on a cloud Linux server. Once you got one, SSH into the server and create the same base directory.

sudo mkdir -p /var/www/repository/

Then use `rsync` to sync the two base directories. Replace 12.34.56.78 with the IP address of your cloud Linux server.

rsync -azP --delete /var/www/repository/ root@12.34.67.78:/var/www/repository/

- `-a`: archive mode
- `-z`: compress file data during the transfer
- `-P`: keep partially transferred files and show progress during transfer
- `--delete`: delete extraneous files from the destination directory.

## Step 6: Set Up HTTP Server

Now we need to set up an HTTP server to expose the repository to the public Internet. We can use Nginx or Apache.

### Nginx

Install Nginx on the cloud server.

sudo apt install nginx

Create a virtual host file for the APT repository.

sudo nano /etc/nginx/conf.d/apt-repository.conf

Add the following lines in this file.

```
server {
  listen 80;
  server_name repo.linuxbabe.com;

  access_log /var/log/nginx/apt-repository.access;
  error_log /var/log/nginx/apt-repository.error;

  location / {
    root /var/www/repository/;
    autoindex on;
  }

  location ~ /(.*)/conf {
    deny all;
  }

  location ~ /(.*)/db {
    deny all;
  }
}
```

Save and close the file. Then test Nginx configuration.

sudo nginx -t

If the test is successful, reload Nginx.

sudo systemctl reload nginx

### Apache

If you prefer to use Apache, then install Apache on the cloud server.

sudo apt install apache2

Create a virtual host file for the APT repository.

sudo nano /etc/apache2/sites-available/apt-repository.conf

Add the following lines in this file.

```
<VirtualHost *:80>
   ServerName repo.example.com
   ErrorDocument 404 /404.html
   DocumentRoot /var/www/repository
   <Directory /var/www/repository/ >
        # We want the user to be able to browse the directory manually
        Options Indexes FollowSymLinks Multiviews
        Require all granted
   </Directory>
   # This syntax supports several repositories, e.g. one for Debian, one for Ubuntu.
   # Replace * with debian, if you intend to support one distribution only.
   <Directory "/var/www/repository/apt/*/db/">
        Require all denied
   </Directory>
   <Directory "/var/www/repository/apt/*/conf/">
        Require all denied
   </Directory>
   <Directory "/var/www/repository/apt/*/incoming/">
        Require all denied
   </Directory>
</VirtualHost>
```

Save and close the file. Then enable this virtual host.

sudo a2ensite apt-repository.conf

Restart Apache.

sudo systemctl restart apache2

## Step 7: Enable HTTPS

To encrypt the HTTP traffic, we can enable HTTPS by installing a free TLS certificate issued from Let’s Encrypt. Run the following command to install Let’s Encrypt client (certbot) on Ubuntu 22.04/20.04.

sudo apt install certbot

If you use **Nginx**, then you also need to install the Certbot Nginx plugin.

sudo apt install python3-certbot-nginx

Next, run the following command to obtain and install TLS certificate.

sudo certbot --webroot -w /var/www/repository -i nginx --agree-tos --redirect --hsts --staple-ocsp --email you@example.com -d repo.example.com

If you use **Apache**, then you need to install the Certbot Apache plugin.

sudo apt install python3-certbot-apache

Next, run the following command to obtain and install TLS certificate.

```
sudo certbot --webroot -w /var/www/repository -i apache --agree-tos --redirect --hsts --staple-ocsp --email you@example.com -d repo.example.com
```

Where:

- `--webroot`: Use the webroot plugin to obtain TLS certificate.
- `-w`: specify the webroot directory.
- `-i nginx`: Use the nginx plugin to install the certificate.
- `-i apache`: Use the Apache plugin to install the certificate
- `--agree-tos`: Agree to terms of service.
- `--redirect`: Force HTTPS by 301 redirect.
- `--hsts`: Add the Strict-Transport-Security header to every HTTP response. Forcing browser to always use TLS for the domain. Defends against SSL/TLS Stripping.
- `--staple-ocsp`: Enables OCSP Stapling. A valid OCSP response is stapled to the certificate that the server offers during TLS.

The certificate should now be obtained and automatically installed.

![enable https on apt repository](https://www.linuxbabe.com/wp-content/uploads/2022/12/enable-https-on-apt-repository.png)

And you can visit your APT repository in the web browser.

![Set Up APT Package Repository on Debian Ubuntu](https://www.linuxbabe.com/wp-content/uploads/2022/12/Set-Up-APT-Package-Repository-on-Debian-Ubuntu.png)

## Step 8: Test

Now we can add the repository on another computer to test if it will work.

Run the following command to import the GPG public key so that APT can verify package integrity during installation.

wget --quiet -O - https://repo.linuxbabe.com/linuxbabe-pubkey.asc | sudo tee /etc/apt/keyrings/linuxbabe-pubkey.asc

Add the repository.

echo "deb [signed-by=/etc/apt/keyrings/linuxbabe-pubkey.asc arch=$( dpkg --print-architecture )] https://repo.linuxbabe.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/firefox-linuxbabe.list

Update repository index.

sudo apt update

Install Firefox. Don’t worry. It won’t overwrite your existing Firefox installation.

sudo apt install firefox-deb

Check where it’s installed.

dpkg -L firefox-deb

You will see that Firefox Deb is installed under the `/opt/firefox-deb/` directory.

![install firefox debian package](https://www.linuxbabe.com/wp-content/uploads/2022/12/install-firefox-debian-package.png)

And you can start it from your application menu, or run the following command.

/opt/firefox-deb/firefox

![firefox web browser without snaps](https://www.linuxbabe.com/wp-content/uploads/2022/12/firefox-web-browser-without-snaps.png)

You might be wondering why I have 3 Firefox icons 🙂 Because I installed the Snap version of Firefox, Firefox ESR (Extended Security Release), and my own Firefox deb package.

If you want to remove the Snap Firefox, run

sudo systemctl disable --now var-snap-firefox-common-host\\x2dhunspell.mount

sudo snap remove firefox

The good thing about using my Firefox deb package is that you can update the browser as soon as Firefox releases a new version. You don’t have to wait for me to push the update via `sudo apt update`.

![firefox debian package self-update](https://www.linuxbabe.com/wp-content/uploads/2022/12/firefox-debian-package-self-update.png)

## How to Remove a Package From Repository

reprepro -V --basedir /var/www/repository/ remove jammy firefox

Then sync the repository.

rsync -azP /var/www/repository/ root@12.34.67.78:/var/www/repository/

https://www.linuxbabe.com/linux-server/set-up-package-repository-debian-ubuntu-server

https://www.debian.org/doc/manuals/maint-guide/