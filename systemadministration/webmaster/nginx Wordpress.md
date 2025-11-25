**Configuring UFW for Nginx:**

If you want to enable both HTTP and HTTPS access, opt for the Nginx Full profile:

```bash
sudo ufw allow 'Nginx Full'
```

However, your requirements might differ:

For HTTPS-only access, choose the Nginx Secure profile:

```bash
sudo ufw allow 'Nginx Secure'
```

HTTP-only access, go with the Nginx HTTP profile:

```bash
sudo ufw allow 'Nginx HTTP'
```

You can create other UFW rules to secure your server and LEMP setup with WordPress, and you should invest time in locking your server down if it is exposed to the public.

### Install MariaDB – Part 2 of LEMP Installation

MariaDB, known for its enhanced performance over MySQL, is the database component in the LEMP stack. If you want to install a specific version of MariaDB from MariaDB.org’s official repositories, refer to guides on [installing MariaDB on Debian](https://linuxcapable.com/how-to-install-mariadb-debian-linux/). This can further optimize your WordPress performance.

To install MariaDB, run:

```bash
sudo apt install mariadb-server mariadb-client
```

After installation, check MariaDB’s status:

```bash
systemctl status mariadb
```

This command displays MariaDB’s service status and any potential errors.

If MariaDB isn’t running, start it with:

```bash
sudo systemctl enable mariadb --now
```

This ensures MariaDB starts with each system reboot, which is vital for a stable LEMP stack and WordPress setup.

### Install PHP, PHP-FPM – Part 3 of LEMP Installation

For a complete LEMP stack, you need to install PHP. PHP acts as the bridge between Nginx and MariaDB, facilitated by PHP-FPM and other essential modules for WordPress.

Note: If you want a specific PHP version tailored to your needs, consult our [guide on installing PHP on Debian](https://linuxcapable.com/how-to-install-php-on-debian-linux/). New users to Linux should for now use the default before installing custom versions.

Run the following command to install PHP, PHP-FPM, and the required modules:

```bash
sudo apt install php php-fpm php-mbstring php-bcmath php-xml php-mysql php-common php-gd php-cli php-curl php-zip php-imagick php-ldap php-intl
```

After the installation, check the PHP service’s status, which is similar to what you did for MariaDB and Nginx. For this example, we’re using PHP 7.4:

```bash
systemctl status php7.4-fpm
```

Note: The PHP-FPM version varies with each stable Debian release. If you’re uncertain about your version, run `php -v` to find out.

## Pre-Installation Configurement For WordPress with LEMP

### Create WordPress Directory Structure

To install WordPress on your Debian LEMP stack, you can either [download](https://wordpress.org/download/) the latest version from the official WordPress.org download page or use the following command to download it directly:

```bash
wget https://wordpress.org/latest.zip
```

Once downloaded, unzip the archive to the /var/www/html directory using the following command:

```bash
sudo unzip latest.zip -d /var/www/html/
```

Next, ensure that WordPress has the correct write permissions by setting the directory owner permissions to the web server user.

This can be done with the following command:

```bash
sudo chown -R www-data:www-data /var/www/html/wordpress/
```

After setting the directory owner permission, you must set the correct permissions for the WordPress folders and files using the following commands:

For folders:

```bash
sudo find /var/www/html/wordpress -type d -exec chmod 755 {} \;
```

And for files:

```bash
sudo find /var/www/html/wordpress -type f -exec chmod 644 {} \;
```

Setting the correct folder and file permissions ensures your WordPress installation is secure and functions correctly.

### Create a Database for WordPress

To run WordPress on your Debian LEMP stack, you must create a database using MariaDB. Access the MariaDB shell as root using the following command:

```bash
sudo mariadb -u root
```

Once in the MariaDB shell, create a new database using the following command:

```bash
CREATE DATABASE WORDPRESSDB;
```

Next, create a new user account for WordPress with the following command:

```bash
CREATE USER 'WPUSER'@localhost IDENTIFIED BY 'PASSWORD';
```

Note: Replace “WPUSER” and “PASSWORD” with your desired username and password.

Finally, assign the newly created user account access to the WordPress website database only using the following command:

```bash
GRANT ALL PRIVILEGES ON WORDPRESSDB.* TO WPUSER@localhost IDENTIFIED BY 'PASSWORD';
```

After creating the user account, flush the privileges to ensure the new changes take effect with the following command:

```bash
FLUSH PRIVILEGES;
```

Lastly, exit the MariaDB shell by typing:

```bash
EXIT;
```

### Set WordPress Configuration Files

Setting up the WordPress configuration files is an essential step in the installation process. This involves renaming the sample wp-config.php file and entering the necessary configuration details.

Navigate to the WordPress directory using the following command:

```bash
cd /var/www/html/wordpress/
```

Copy the wp-config-sample.php to wp-config.php using the following command:

```bash
sudo cp wp-config-sample.php wp-config.php
```

Using a text editor, bring up the newly copied wp-config.php file:

```bash
sudo nano wp-config.php
```

Next, enter the database name, user account with a password, and host IP address if necessary.

```sql
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */ 

define( 'DB_NAME', 'WORDPRESSDB' );                 <--------------- change this

/* MySQL database username */ 

define( 'DB_USER', 'WPUSER );                               <--------------- change this

/* MySQL database password */

define( 'DB_PASSWORD', 'PASSWORD' );             <--------------- change this

/* MySQL hostname, change the IP here if external DB set up */ 

define( 'DB_HOST', 'localhost' );

/* Database Charset to use in creating database tables. */

define( 'DB_CHARSET', 'utf8' );

/* The Database Collate type. Don't change this if in doubt. */

define( 'DB_COLLATE', '' );
```

In addition to these settings, you can also add the following to the wp-config.php file to improve WordPress management:

```
/** ## Save files direct method ## */
define( 'FS_METHOD', 'direct' );

/** ## Increase memory limit, 256MB is recommended ## */
define('WP_MEMORY_LIMIT', '256M');
```

Your dedicated server’s or VPS’s memory limit can vary depending on your system’s capacity. You can increase or decrease the 256 MB memory limit in small increments, such as 128 MB, 256 MB, 512 MB, etc.

Note: It is important to note that it is recommended to only make small adjustments to the memory limit for optimal performance and stability.

#### Integrating Security Salt Keys

To embed the freshly generated security salt keys into your wp-config.php file, open the file in a text editor:

```bash
sudo nano /var/www/html/wordpress/wp-config.php
```

Now, identify the lines in the wp-config.php file corresponding to the sample keys. Once located, replace each sample key in the wp-config.php file with your newly generated keys. After making the necessary replacements, ensure you save and close the file.

If you’re using the nano editor, save by pressing “CTRL+X” followed by “Y”.

### Nginx Server Block Configuration for WordPress LEMP Setup

Setting up the Nginx server block correctly is vital for a seamless WordPress installation via the web UI. It’s essential to get the “try_files $uri $uri/ /index.php?$args;” directive right. Leaving out the “?$args” can interfere with WordPress’s REST API. To avoid potential hiccups during the installation, follow these instructions closely.

Create a new server configuration file for your WordPress installation on Nginx. Replace “example.com” with your actual domain name in the following command:

```bash
sudo nano /etc/nginx/sites-available/example.com.conf
```

For Nginx to work with PHP, you must include the “location ~ .php$” in the server block configuration file. Here’s a sample configuration you can use as a reference.

Ensure you adjust the root path and domain names to fit your setup:

```nginx
server {
  listen 80;
  listen [::]:80;
  server_name www.example.com example.com;
  root /var/www/html/wordpress;
  index index.php index.html index.htm index.nginx-debian.html;

  location / {
    try_files $uri $uri/ /index.php?$args;
  }

  location ~* /wp-sitemap.*\.xml {
    try_files $uri $uri/ /index.php$is_args$args;
  }

  client_max_body_size 100M;

  location ~ \.php$ {
    fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
    fastcgi_buffer_size 128k;
    fastcgi_buffers 4 128k;
    fastcgi_intercept_errors on;
  }

  gzip on;
  gzip_comp_level 6;
  gzip_min_length 1000;
  gzip_proxied any;
  gzip_disable "msie6";
  gzip_types application/atom+xml application/geo+json application/javascript application/x-javascript application/json application/ld+json application/manifest+json application/rdf+xml application/rss+xml application/xhtml+xml application/xml font/eot font/otf font/ttf image/svg+xml text/css text/javascript text/plain text/xml;

  location ~* \.(?:css(\.map)?|js(\.map)?|jpe?g|png|gif|ico|cur|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm|mpe?g|avi|ogv|flv|wmv)$ {
    expires 90d;
    access_log off;
  }

  location ~* \.(?:svgz?|ttf|ttc|otf|eot|woff2?)$ {
    add_header Access-Control-Allow-Origin "*";
    expires 90d;
    access_log off;
  }

  location ~ /\.ht {
    access_log off;
    log_not_found off;
    deny all;
  }
}
```

Remember, adjust the Nginx configuration file accordingly if you’ve installed a different PHP or PHP-FPM version or your Debian version defaults to another PHP version.

For example, for PHP-FPM 8.2, change the line fastcgi_pass unix:/run/php/php8.1-fpm.sock; to fastcgi_pass unix:/run/php/php8.2-fpm.sock;. It is essential to match the version in the configuration with the one on your system for smooth functionality.

### Understanding the WordPress Nginx Server Block

For those new to setting up Nginx and WordPress, here’s a breakdown of the server block example:

#### Basic Server Settings:

- These settings define the foundational aspects of the server block, such as the IP address, port for Nginx to listen on, and server names.
- The root directive points to the primary directory containing the website files.
- The index directive instructs Nginx on identifying index files when serving the site.

#### Location Settings:

- These settings include various location blocks that dictate how Nginx processes requests for different URLs.
- The initial location block manages requests to the site’s root URL, utilizing the try_files directive.
- The subsequent location block processes requests specifically for the WordPress sitemap.xml file.

#### PHP Handling Settings:

- These settings determine how Nginx processes PHP files.
- The fastcgi_pass directive points to the PHP-FPM socket file’s location.
- The fastcgi_param directive assigns the SCRIPT_FILENAME parameter’s value to the requested PHP file’s location.
- The include directives pull in additional configuration files for the FastCGI module.
- Directives like fastcgi_buffer_size and fastcgi_buffers designate the buffer size for data transfer between Nginx and PHP-FPM.
- The fastcgi_intercept_errors directive empowers Nginx to capture and manage PHP errors.

#### Gzip Compression Settings:

- These settings configure Gzip compression, reducing the file size delivered to the client.
- The gzip directive activates Gzip compression.
- Directives like gzip_comp_level and gzip_min_length determine the compression level and the minimum file size for compression, respectively.
- The gzip_proxied directive identifies which request types undergo compression.
- The gzip_types directive enumerates the MIME types eligible for compression.

#### File Caching Settings:

- These settings optimize caching for static files, enhancing website speed.
- The initial location block establishes the expiration duration for asset and media files.
- The subsequent location block sets the expiration for font and SVG files.
- Directives such as access_log and log_not_found govern the request logging.
- The add_header directive appends the Access-Control-Allow-Origin header, permitting font and SVG loading from external domains.

#### .htaccess File Blocking:

- This setting restricts access to files starting with .ht, typically sensitive server configuration files.

### Setting Up the Nginx Server Block with a Symbolic Link

To wrap up the Nginx server block configuration, you must activate the configuration file from the “sites-available” directory. This is achieved by creating a symbolic link to the “sites-enabled” directory.

Execute the command below, ensuring you replace “example.com.conf” with your configuration file’s name:

```bash
sudo ln -s /etc/nginx/sites-available/example.com.conf /etc/nginx/sites-enabled/
```

This command establishes a symbolic link between the directories, granting Nginx access to the configuration file. After setting this up, validate the configuration with:

```bash
sudo nginx -t
```

If the test returns no errors, restart Nginx to apply the server block changes:

```bash
sudo systemctl restart nginx
```

With these steps completed, your WordPress site should now be accessible via Nginx.

### Configuring PHP.ini for Optimal WordPress Performance

Tuning your PHP settings is crucial for achieving the best performance with WordPress. To handle media files efficiently in WordPress, consider increasing the maximum upload size, post size, and memory limit. You might also need to tweak the maximum execution time and input variables to prevent potential issues.

To access your php.ini file, use the terminal. Remember, the file’s location might differ based on your PHP version:

```bash
sudo nano /etc/php/8.0/fpm/php.ini
```

```bash
sudo nano /etc/php/8.1/fpm/php.ini
```

```bash
sudo nano /etc/php/8.2/fpm/php.ini
```

```bash
sudo nano /etc/php/8.3/fpm/php.ini
```

To tailor the PHP settings, find and adjust the following lines in your php.ini file:

```
##increase this to the maximum file size you want to upload, recommended 50 to 100MB## 
 upload_max_filesize = 100M

##increase this to the maximum post size you want to allow, recommended 50 to 100MB##
 post_max_size = 100M

##increase this to the maximum execution time, recommended 150 to 300 seconds##
 max_execution_time = 300

##increase this to the maximum GET/POST/COOKIE input variables, recommended 5000 to 10000##
max_input_vars = 5000

##increase this to the maximum memory limit, recommended 256MB or 512MB. Note that you should ensure your system has enough RAM before raising this.##
memory_limit = 256M
```

After modifying your PHP settings, it’s vital to restart the PHP-FPM server. This ensures the new configurations are active, allowing your WordPress site to operate at its best.

### Increase Nginx Server Client Max Body Size

To accommodate larger file uploads on your WordPress site, you’ll need to tweak the Nginx server block. This ensures that Nginx can handle larger HTTP request bodies, which is essential when dealing with sizable file uploads.

#### Modifying the Nginx Server Block

Open your server block configuration file and insert the following line:

```bash
##set to the maximum upload size you set in upload_max_filesize.##
client_max_body_size – <size>
```

Ensure that the value for client_max_body_size aligns with the upload_max_filesize you configured in your PHP settings.

#### Restarting PHP-FPM

After adjusting the PHP settings for optimal WordPress performance, including upload size, post size, and memory limit, it’s crucial to restart the PHP-FPM server for the changes to take effect. The exact command to restart the server depends on your PHP version. If you’re unsure about your PHP version, consult your system’s documentation.

For different PHP versions, use the corresponding commands to restart PHP-FPM:

```bash
sudo systemctl restart php8.0-fpm
```

```bash
sudo systemctl restart php8.1-fpm
```

```bash
sudo systemctl restart php8.2-fpm
```

```bash
sudo systemctl restart php8.3-fpm
```

## Install WordPress Front-end

After finalizing the backend setup and configuration, launching the WordPress front end on your domain is time. Start the installation by heading to your domain, prefixed by “https://” or “http://.” Alternatively, you can directly access “https://www.yoursite.com/wp-admin/install.php.”

This URL directs you to the front-end installation wizard.

### Step 1: Select WordPress Language

Select your desired language and click **“Continue.”**

### Step 2: Create Admin User For WordPress

Next, you’ll land on a page prompting you to input your site title, username, password, and the main admin’s email address for the WordPress site.

For security reasons, choose a robust password and provide a valid email address. Remember that you can modify other settings later within the WordPress settings panel.

For those developing their site and wishing to keep it private from search engines like Google or Bing, there’s an option to “strongly discourage search engines from indexing.”

### Step 3: Proceed and Click Install WordPress Button

After filling out your details and preferences, hit the **“Install WordPress”** button. A successful installation will redirect you to the login page.

![WordPress web UI installation success login page on Debian with LEMP](https://linuxcapable.com/wp-content/uploads/2024/08/wordpress-install-on-debian-with-lemp-wordpress-web-ui-install-success-login-page-proceed.png)

### Step 4: Proceed to log in on the WordPress Admin Page

Input your login details and press “Log in.” This action will usher you into the WordPress dashboard, where you can craft or import your website.

### Step 5: View and Adjust the WordPress site via WordPress Admin

The WordPress dashboard is your command center. Here, you can draft new posts, design pages, handle themes and plugins, and tailor your site’s look, content, and operations.

With its user-friendly interface, the dashboard empowers you to swiftly establish your website, enabling you to design a captivating and professional site with minimal effort.

## Additional Tips For WordPress with Nginx

### Securing WordPress and Nginx with Let’s Encrypt SSL Certificate

Enhancing your web server’s security is paramount, and one effective way to achieve this is by running Nginx on HTTPS using an SSL certificate. Let’s Encrypt offers a free, automated, and open certificate authority, making setting up SSL certificates for your Nginx server easier.

#### Installing Certbot

Start by installing the certbot package with the command:

```bash
sudo apt install python3-certbot-nginx
```

#### Generating the SSL Certificate

Once you’ve installed the certbot package, generate your SSL certificate with the following:

```bash
sudo certbot --nginx --agree-tos --redirect --hsts --staple-ocsp --email you@example.com -d www.example.com
```

Certbot will prompt you to input your email and domain name during this process. You’ll also have the option to receive emails from the EFF. Decide whether to opt-in based on your preferences.

After installing the certificate, your website’s URL will switch from HTTP to HTTPS. Visitors accessing the old HTTP URL will automatically be redirected to the new HTTPS URL. This configuration ensures HTTPS 301 redirects, a Strict-Transport-Security header, and OCSP Stapling for top-tier security.

#### Setting Up Automatic Certificate Renewal

To keep your SSL certificate valid, set up a cron job for its automatic renewal. Certbot offers a script for this. Before finalizing the setup, run a dry run test:

```bash
sudo certbot renew --dry-run
```

Access the crontab configuration, enter:

```bash
sudo crontab -e
```

To automatically renew your SSL certificate, schedule it using a cron job with the following command:

```bash
00 00 */1 * * /usr/sbin/certbot-auto renew
```

This will attempt to renew the certificate every day at midnight.

### Resolving PHP Session Errors

Encountering session-saving issues, especially when using certain plugins? The root of the problem might be incorrect user permissions in the /var/lib/php/sessions/ directory. But don’t worry; you can address this with a straightforward command.

#### Adjusting Directory Permissions

Run the command below to set the correct permissions:

```bash
sudo chown -R www-data:www-data /var/lib/php/sessions/
```

This command sets the www-data user and group as the owners of the sessions directory. As a result, WordPress can write session data without any hitches. This adjustment is vital for the seamless operation of your WordPress site, mainly if you’re using plugins that handle automated tasks, such as posting to social media.

Addressing PHP session errors is key to boosting your website’s performance and improving the user experience.

### Resolving HTTPS Redirect Loop in WordPress

If your WordPress site finds itself trapped in a redirect loop after activating HTTPS, it’s likely because WordPress keeps trying to redirect to the secure HTTPS version, but the loop never completes. To tackle this, you can modify your wp-config.php file with specific lines of code.

#### Modifying the wp-config.php File

Insert the following lines into your wp-config.php:

```
define('FORCE_SSL_ADMIN', true);

if (strpos($_SERVER['HTTP_X_FORWARDED_PROTO'], 'https') !== false) {
    $_SERVER['HTTPS'] = 'on';
}
```

Here’s a breakdown of what each line does:

- The first line sets the FORCE_SSL_ADMIN constant to true, ensuring all admin pages use HTTPS.
- The subsequent code checks if the HTTP_X_FORWARDED_PROTO header contains the term “https.” If it finds a match, it designates the HTTPS server variable as “on.” This action informs WordPress that the connection is secure.

Integrating these lines into your wp-config.php file, you should be able to break free from the HTTPS redirect loop and ensure your WordPress site operates smoothly with its new secure connection.

### Fix Domain Name Redirect Loop

A redirect loop in your WordPress site can sometimes stem from a mismatch between the domain name specified in your wp-config.php file and your website’s domain name. To address this, you’ll need to verify and possibly adjust the domain name in the configuration.

#### Checking the wp-config.php File

Inspect the following lines in your wp-config.php:

```
define('WP_HOME','http://example.com');
define('WP_SITEURL','http://example.com');
```

If the domain name doesn’t align with your website’s domain, correct it accordingly.