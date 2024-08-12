# Joomla
Joomla is a popular, open-source content management system (CMS) that enables users to create and manage websites easily. Known for its flexibility, extensive plugin ecosystem, and user-friendly interface, it supports various types of websites, from blogs to complex e-commerce sites.


![](https://www.appvizer.com/media/application/450/screenshot/2474/4317.png)


To install Joomla using Nginx, MySQL, and an SSL certificate on Ubuntu 22.04, follow these steps:

### 1. **Update and Install Required Packages**

First, update your system and install the required packages:

```bash
sudo apt update
sudo apt upgrade
sudo apt install nginx mysql-server php-fpm php-mysql php-xml php-mbstring unzip curl
```

### 2. **Set Up MySQL Database**

Log in to MySQL and create a database and user for Joomla:

```bash
sudo mysql -u root -p
```

Inside the MySQL shell:

```sql
CREATE DATABASE joomla_db;
CREATE USER 'joomla_user'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON joomla_db.* TO 'joomla_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 3. **Download and Install Joomla**

Navigate to the web root directory and download Joomla:

```bash
cd /var/www/html
sudo curl -O https://downloads.joomla.org/cms/joomla4/4-4-7/Joomla_4.4.7-Stable-Full_Package.zip
sudo unzip Joomla_4.4.7-Stable-Full_Package.zip
sudo mv Joomla_4.4.7-Stable-Full_Package/* joomla/
sudo chown -R www-data:www-data joomla
sudo chmod -R 755 joomla
```

### 4. **Configure Nginx for Joomla**

Create a new Nginx server block configuration for Joomla:

```bash
sudo nano /etc/nginx/sites-available/joomla
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your_domain.com;
    root /var/www/html/joomla;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Link this configuration to `sites-enabled`:

```bash
sudo ln -s /etc/nginx/sites-available/joomla /etc/nginx/sites-enabled/
```

Test the Nginx configuration:

```bash
sudo nginx -t
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

### 5. **Set Up SSL Certificate**

You can use Let's Encrypt to obtain a free SSL certificate. Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx
```

Obtain and install the SSL certificate:

```bash
sudo certbot --nginx -d your_domain.com
```

Follow the prompts to complete the certificate setup.

### 6. **Complete Joomla Installation**

Navigate to your domain in a web browser:

```
http://your_domain.com
```

Follow the Joomla installation wizard to complete the setup. Enter the database details and other required information.

### 7. **Secure Your Joomla Installation**

After installation, make sure to:

- Remove the `installation` directory from Joomla:

  ```bash
  sudo rm -rf /var/www/html/joomla/installation
  ```

- Ensure proper file permissions and ownership:

  ```bash
  sudo chown -R www-data:www-data /var/www/html/joomla
  sudo find /var/www/html/joomla -type d -exec chmod 755 {} \;
  sudo find /var/www/html/joomla -type f -exec chmod 644 {} \;
  ```

Now, your Joomla site should be up and running with Nginx, MySQL, and SSL on Ubuntu 22.04!
