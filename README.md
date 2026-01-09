# Create Your Own Self-Hosted Cloud Storage

## Benefits
- Enhanced privacy and security
- Cost-effective
- Scalable
- Greater control and customization

## Prerequisites
- A computer running Ubuntu or any Debian-based distribution.
- Verify if your ISP allows port forwarding [Optional]
- Own a domain [Optional]

## Initial Server Setup

### Update Your Linux Distribution
```bash
sudo apt update
sudo apt full-upgrade
sudo apt autoremove
```

### Install Necessary Packages
```bash
sudo apt install apache2 libapache2-mod-php neovim wget mariadb-server php php-apcu php-bcmath php-bz2 php-cli php-common php-curl php-gd php-gmp php-imagick php-intl php-mbstring php-mysql php-zip php-xml unzip nmap ffmpeg
```

### Update Your Hostname (Optional)
- Modify the file to assign an appropriate hostname or domain name to your server.

```bash
sudo nvim /etc/hostname
```
![hostname.png](pictures/hostname.png)

```bash
sudo nvim /etc/hosts
```
![hosts.png](pictures/hosts.png)

- Restart your server to apply the changes.

```bash
sudo reboot
```

## Downloading Nextcloud
- Download the Nextcloud ZIP file.

```bash
wget https://download.nextcloud.com/server/releases/latest.zip
```

## MariaDB Setup
- Check the status of the `mariadb` service.

```bash
systemctl status mariadb
```

- Run the secure installation script.

```bash
sudo mysql_secure_installation
```

- Follow the prompts with the provided inputs:

```
Password prompt --> Press Enter
Switch to unix_socket_auth [Y/n] --> n
Change the root password [Y/n] --> Y
Enter your secure password:
Remove anonymous users [Y/n] --> Y
Disallow root login remotely [Y/n] --> Y
Remove test database and access to it [Y/n] --> Y
Reload privilege tables now [Y/n] --> Y
```

## Setting Up the Nextcloud Database
- Access MariaDB.

```bash
sudo mariadb
```

- Create the database.

```sql
CREATE DATABASE nextcloud;
```

- Set up permissions.

```sql
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY 'mypassword';
FLUSH PRIVILEGES;
```

- Exit by pressing `CTRL+D`.

## Deploy Nextcloud Files

- Enable the necessary PHP extensions.

```bash
sudo phpenmod bcmath bz2 gmp imagick intl
```

- Unzip the Nextcloud file.

```bash
unzip latest.zip
```

- Move the files to the serving location and set the appropriate permissions.

```bash
mv nextcloud your.domain.name
sudo chown -R www-data:www-data your.domain.name
sudo mv your.domain.name /var/www
```

- Disable the default Apache site.

```bash
sudo a2dissite 000-default.conf
```

## Configure Nextcloud Host
- Create an Apache config file for serving Nextcloud.

```bash
sudo nvim /etc/apache2/sites-available/your.domain.name.conf
```

- Insert the following content into the file, modifying `your.domain.name` as needed:

```bash
<VirtualHost *:80>
    DocumentRoot "/var/www/your.domain.name"
    ServerName your.domain.name

    <Directory "/var/www/your.domain.name/">
        Options MultiViews FollowSymlinks
        AllowOverride All
        Order allow,deny
        Allow from all
    </Directory>

    TransferLog /var/log/apache2/your.domain.name_access.log
    ErrorLog /var/log/apache2/your.domain.name_error.log
</VirtualHost>
```

- Enable the site.

```bash
sudo a2ensite your.domain.name.conf
```

## PHP Configuration
- Edit the PHP configuration file (replace 8.3 with your PHP version if different).

```bash
sudo nano /etc/php/8.3/apache2/php.ini
```

- Locate and modify the following parameters:

```php
memory_limit = 3G # Increase if you have more RAM
upload_max_filesize = 50G # Adjust based on your needs
max_execution_time = 3600
post_max_size = 50G # Adjust based on your needs
date.timezone = Europe/London # Adjust your timezone
opcache.enable=1
opcache.interned_strings_buffer=128
opcache.max_accelerated_files=10000
opcache.memory_consumption=128
opcache.save_comments=1
opcache.revalidate_freq=1
```

[Timezones](https://www.php.net/manual/en/timezones.php)

- Enable PHP modules for Apache.

```bash
sudo a2enmod dir env headers mime rewrite ssl
```

- Restart Apache.

```bash
sudo systemctl restart apache2
```

## Network Configuration
To access Nextcloud securely, you should configure your network and domain before completing the installation.

### Port Forwarding (For Home Servers)
- Enable port forwarding on your router for ports 80 (HTTP) and 443 (HTTPS) to your server's IP address.
- Example configuration for a Movistar router:

1. Find your private IP using:

```bash
ifconfig -a
```

![ifconfig.png](pictures/ifconfig.png)

2. Configure port forwarding in your router settings.
3. Verify ports are open:

```bash
sudo nmap -n -PN -sT -sU -p80,443 {IP}
```

### Configure Your Domain (Optional but Recommended)

- Purchase or obtain a domain from providers like Namecheap, Cloudflare, etc.
- In your DNS settings, create an `A` record pointing to your public IP.

![namecheap_host.png](pictures/namecheap_host.png)

## TLS Certificate (HTTPS)

- Secure your installation with a free certificate from Let's Encrypt.
- Install `snapd` and `certbot`:

```bash
sudo apt install snapd
sudo snap install core && sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

- Obtain and install your certificates:

```bash
sudo certbot --apache
```

## Set Up Nextcloud Web Server
- Access your Nextcloud instance via your domain (e.g., `https://your.domain.name`).
- Upon first access, you will see a setup page.

![admin.png](pictures/admin.png)

- Enter the following information:

```txt
Username --> Your desired username
Password --> Set a secure password
Data Folder --> Leave as default
Database user --> nextcloud
Database password --> Your database password
Database name --> nextcloud
```

- Install the recommended apps.

## Post-Installation Optimization

### Background Jobs
- Set up a cron job for the `www-data` user to execute background tasks.

```bash
sudo crontab -u www-data -e
```

- Add the following line to the file:

```cron
*/5  *  *  *  * php -f /var/www/your.domain.name/cron.php
```

### Memory Caching
- Enable APCu for local caching by editing the config file.

```bash
sudo nvim /var/www/your.domain.name/config/config.php
```

- Add the following line to the configuration array:

```php
'memcache.local' => '\OC\Memcache\APCu',
```

### Trusted Domains (If Needed)
- If you change your domain or IP later, edit `config.php`:

```bash
sudo nvim /var/www/your.domain.name/config/config.php
```

- Add your new domain or IP to the `trusted_domains` array.

![trusted.png](pictures/trusted.png)

## Troubleshooting
- Search online forums for solutions.

## Resources
- [Nextcloud Documentation](https://docs.nextcloud.com/)
- [Nextcloud Community](https://help.nextcloud.com/)

**Please submit a pull request for any errors or suggestions.**