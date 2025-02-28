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
sudo apt install neovim wget mariadb-server php php-apcu php-bcmath php-cli php-common php-curl php-gd php-gmp php-imagick php-intl php-mbstring php-mysql php-zip php-xml unzip nmap
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

## Apache Web Server

- Enable the necessary PHP extensions.

```bash
sudo phpenmod bcmath gmp imagick intl
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
- Edit the PHP configuration file.

```bash
sudo nano /etc/php/8.1/apache2/php.ini
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

## Set Up Nextcloud Web Server
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

## Optional: External Access

- To access Nextcloud from outside your private network, enable port forwarding on your router.
- Example configuration for a Movistar router:

1. Find your private IP using:

```bash
ifconfig -a
```

![ifconfig.png](pictures/ifconfig.png)

2. Configure port forwarding in your router settings, enabling port 80 for your IP.
3. Verify port 80 is open:

```bash
sudo nmap -n -PN -sT -sU -p80 {IP}
```

4. Check if the Nextcloud page is accessible by entering your public IP in a web browser.

![untrusted.png](pictures/untrusted.png)

### Extra Configuration
- To access Nextcloud via your public IP, edit the `config.php` file:

```bash
sudo nvim /var/www/your.domain.name/config/config.php
```

- Add your public IP to the `trusted_domains` array.

![trusted.png](pictures/trusted.png)

## Configure Your Domain

- Purchase or obtain a domain from providers like:
  - [Namecheap](https://www.namecheap.com)
  - [Cloudflare](https://www.cloudflare.com)
  - [Squarespace](https://domains.squarespace.com)
  - [Name.com](https://www.name.com)

### Set Up Namecheap
1. In Advanced DNS settings, create a host record with your public IP.
2. Edit the `config.php` file to add your domain to the `trusted_domains` array.

![namecheap_host.png](pictures/namecheap_host.png)
![domain_trusted.png](pictures/domain_trusted.png)

## TLS Certificate

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

## Troubleshooting
- Search online forums for solutions.
- For further assistance, contact me via [email](mailto:boxy_lecturer710@simplelogin.com).

## Resources
- [Nextcloud Documentation](https://docs.nextcloud.com/)
- [Nextcloud Community](https://help.nextcloud.com/)

**Please submit a pull request for any errors or suggestions.**