# ngix-laravel

Setup Laravel with Apache server and MySQL database on your domain.

## Ngnix Setup

### Step 1: Install Nginx

#### Update your package index:

```bash
sudo apt update
```

#### Install Nginx:

```bash
sudo apt install nginx
```

#### Start Nginx:

```bash
sudo systemctl start nginx
```

#### Enable Nginx to start at boot:

```bash
sudo systemctl enable nginx
```

#### Check Nginx status:

```bash
sudo systemctl status nginx
```

You should see output that indicates Nginx is running.

### Step 2: Adjust the Firewall (Optional)

If you're using ufw (Uncomplicated Firewall), you need to allow Nginx:

#### Allow Nginx HTTP and HTTPS traffic:

```bash
sudo ufw allow 'Nginx Full'
```

#### Enable the firewall (if not already enabled):

```bash
sudo ufw enable
```

#### Check the firewall status:

```bash
sudo ufw status
```

### Step 3: Basic Configuration

Default Server Block (Optional) Nginx's default server block is located in `/etc/nginx/sites-available/default.`

#### You can edit this file to change the server's root directory, server name, and other settings:

```bash
sudo nano /etc/nginx/sites-available/default
```

Here's an example of what it might look like:

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    server_name your_domain.com www.your_domain.com;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

#### Test Nginx Configuration: After making changes, test your configuration to ensure there are no syntax errors:

```bash
sudo nginx -t
```

#### Reload Nginx: If the test is successful, reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

### Step 4: Setting Up Additional Server Blocks (Optional)

To host multiple sites on your server, you can set up additional server blocks.

#### Create a new server block file:

```bash
sudo nano /etc/nginx/sites-available/your_domain.com
```

Add your server block configuration:

```nginx
server {
    listen 80;
    server_name your_domain.com www.your_domain.com;

    root /var/www/your_domain.com/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

#### Enable the new server block: Link it to the sites-enabled directory:

```bash
sudo ln -s /etc/nginx/sites-available/your_domain.com /etc/nginx/sites-enabled/
```

#### Test and reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### Step 5: Monitoring and Maintenance

Check Nginx logs:

-   Access logs: `/var/log/nginx/access.log`
-   Error logs: `/var/log/nginx/error.log`

#### Reload or Restart Nginx after changes:

```bash
sudo systemctl reload nginx
sudo systemctl restart nginx
```

This is a basic setup guide, and Nginx has many advanced features like reverse proxying, load balancing, caching, etc., that you can explore as needed.

## Setup Laravel

### Step 1: Clone the Laravel Repository

Clone your Laravel repository:

```bash
git clone --depth 1 <repository_url> <destination_directory>
```

-   <repository_url>: The URL of your Laravel repository.
-   <destination_directory>: Directory where you want to clone the repository.

### Step 2: Install PHP and Required Extensions

Install PHP and necessary extensions:

```bash
sudo apt update
sudo apt install php php-cli php-mysql libapache2-mod-php php-xml php-mbstring
```

## Install MySQL

### Step 1: Install MySQL

```bash
sudo apt update
sudo apt install mysql-server
```

### Step 2: Secure MySQL Installation

Run the MySQL secure installation script to improve security:

```bash
sudo mysql_secure_installation
```

### Step 3: Create a MySQL Database and User for Laravel

Log into MySQL as the root user:

```bash
sudo mysql -u root -p
```

Create a new database and user, and grant privileges:

```sql
CREATE DATABASE your_database_name;
CREATE USER 'your_database_user'@'localhost' IDENTIFIED BY 'your_database_password';
GRANT ALL PRIVILEGES ON your_database_name.* TO 'your_database_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Step 4: Update Laravel Configuration

Update the .env file in your Laravel project to include the database credentials:

```bash
sudo nano /var/www/html/<destination_directory>/.env
```

Ensure the following lines are updated with your database information:

```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_database_name
DB_USERNAME=your_database_user
DB_PASSWORD=your_database_password
```

### Step 5: Install Laravel Dependencies

Navigate to your Laravel project directory and install dependencies:

```bash
cd /var/www/html/<destination_directory>
composer install
```

### Step 6: Generate Application Key

Generate a new application key:

```bash
php artisan key:generate
```

### Step 7: Migrate Database (if needed)

Run migrations to set up your database schema:

```bash
php artisan migrate
```

## Configure Apache

To configure Apache to serve a website based on a domain name, you'll need to set up a Virtual Host. Here's how you can do it on Ubuntu:

### Step 1: Enable the rewrite module (optional but recommended)

This module allows URL rewriting, which is often needed for domain-based routing.

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### Step 2: Create a Virtual Host Configuration File

#### Navigate to the sites-available directory:

```bash
cd /etc/apache2/sites-available/
```

#### Create a new configuration file for your domain:

-   #### Replace yourdomain.com with your actual domain name.

```bash
sudo nano yourdomain.com.conf
```

-   #### Add the following content to the file:

```apache
<VirtualHost *:80>
    ServerName apache-laravel.bimash.com.np

    ProxyPass /  http://localhost:8000/
    ProxyPassReverse /  http://localhost:8000/

    <Proxy *>
        Require all granted
    </Proxy>

    # Optional: Log directives for debugging
    ErrorLog ${APACHE_LOG_DIR}/proxy_error.log
    CustomLog ${APACHE_LOG_DIR}/proxy_access.log combined
</VirtualHost>

```

-   #### Enable Required Apache Modules

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
```

-   #### Save and Enable the Virtual Host

```bash
sudo a2ensite apache-laravel.bimash.com.np.conf
```

### Step 3: Disable Conflicting Sites

If there’s a default site that could conflict, disable it:

```bash
sudo a2dissite 000-default.conf
```

### Step 4: Reload Apache

Apply the configuration changes by reloading Apache:

```bash
sudo systemctl reload apache2
```

### Step 5: Check DNS

Ensure that the DNS for apache-laravel.bimash.com.np points to your server’s IP address.

### Step 6: Test the Configuration

Open a web browser and go to http://yourdomain.com. You should see the "Welcome to yourdomain.com" message you added in the index.html file.
If you need to use SSL (HTTPS), you can obtain and configure an SSL certificate using Let's Encrypt with Certbot. This is often required for modern web standards.

## Supervisor/Systemd Configuration

To run php artisan serve in the background, you can use either systemd or supervisor. Below are instructions for both methods:

## Using systemd

#### Step 1: Create or Edit the Service File

Create or edit the systemd service file:

```bash
sudo nano /etc/systemd/system/laravel-serve.service
```

#### Step 2: Add the Service Configuration

Add the following configuration:

```ini
[Unit]
Description=Laravel Development Server
After=network.target

[Service]
User=azureuser
Group=azureuser
WorkingDirectory=/home/azureuser/apache/apache-laravel/
ExecStart=/usr/bin/php /home/azureuser/apache/apache-laravel/artisan serve --host=0.0.0.0 --port=8000
Restart=always
RestartSec=5
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=laravel-serve

[Install]
WantedBy=multi-user.target
```

-   User and Group: Set to azureuser.
-   WorkingDirectory: Points to your Laravel project.
-   ExecStart: Runs the php artisan serve command.

#### Step 3:Reload systemd

```bash
sudo systemctl daemon-reload
```

#### Start the Service

```bash
sudo systemctl start laravel-serve
```

#### Enable the Service at Boot

```bash
sudo systemctl enable laravel-serve
```

#### Check the Status

```bash
sudo systemctl status laravel-serve
```

## Using supervisor

### Step 1: Install Supervisor

If it's not already installed, install Supervisor:

```bash
sudo apt-get update
sudo apt-get install supervisor
```

### Step 2: Create a Supervisor Configuration File

Create a configuration file for your Laravel application:

```bash
sudo nano /etc/supervisor/conf.d/laravel-serve.conf
```

### Step 3: Add the Supervisor Configuration

Add the following configuration:

```ini
[program:laravel-serve]
command=/usr/bin/php /home/azureuser/apache/apache-laravel/artisan serve --host=0.0.0.0 --port=8000
directory=/home/azureuser/apache/apache-laravel/
autostart=true
autorestart=true
user=azureuser
stdout_logfile=/var/log/laravel-serve.log
stderr_logfile=/var/log/laravel-serve.err
```

-   command: The command to start the Laravel server.
-   directory: Working directory for the command.
-   autostart and autorestart: Ensure the service starts on boot and restarts on failure.
-   user: The user to run the command as.
-   stdout_logfile and stderr_logfile: Paths to log files.
-   Update Supervisor and Start the Service

### Step 4: Update Supervisor to include the new configuration and start the service:

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-serve
```

### Step 5: Check the Status

Verify that the service is running:

```bash
sudo supervisorctl status
```

## Notes

-   systemd: Ideal for production or more integrated service management.
-   supervisor: Useful for managing multiple processes or in development environments.
    Both methods will keep your Laravel application running in the background. Choose the one that best fits your needs.

## Manual Deployment

### Step 1: Pull the Latest Changes

Navigate to your Laravel project directory and pull the latest changes:

```bash
cd /path/to/your/apache-laravel/
git pull origin main
```

Adjust the branch name (main in this case) as needed.

### Step 2: Clear and Cache Configurations

Run Laravel's artisan commands to clear any cached configurations or views:

```bash
php artisan config:cache
php artisan route:cache
php artisan view:clear
```

### Step 3: Reload Apache

To apply any changes to the Apache configuration, reload Apache:

```bash
sudo systemctl reload apache2
```

### Step 4: Check for Laravel Queue Workers or Background Jobs

If your application uses queue workers or background jobs, you might need to restart them. You can restart the Laravel queue workers with:

```bash
php artisan queue:restart
```

If you're using Supervisor or another process manager, restart those services as well.

### Step 5: Verify Changes

After reloading Apache and clearing the Laravel cache, visit your application’s URL to verify that the changes have been applied correctly.

## CI/CD Pipeline Deployment

```yaml
name: 🚀 Deploy Laravel
on:
    push:
        branches:
            - main

jobs:
    web-deploy:
        name: 🎉 Deploy
        runs-on: ubuntu-latest

        steps:
            - name: Checkout code
              uses: actions/checkout@v2

            - name: Set up PHP
              uses: shivammathur/setup-php@v2
              with:
                  php-version: "8.1.2"

            - name: Install Composer dependencies
              run: composer install --no-interaction --no-suggest

            - name: Set up SSH
              uses: webfactory/ssh-agent@v0.7.0
              with:
                  ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

            - name: Deploy to Azure VM
              run: |
                  ssh -o StrictHostKeyChecking=no azureuser@20.51.207.28 << 'EOF'
                  cd apache/apache-laravel/
                  git pull origin main || { echo 'Git pull failed'; exit 1; }
                  php artisan config:cache || { echo 'Config cache failed'; exit 1; }
                  php artisan route:cache || { echo 'Route cache failed'; exit 1; }
                  php artisan view:clear || { echo 'View clear failed'; exit 1; }
                  sudo systemctl reload apache2 || { echo 'Apache reload failed'; exit 1; }
                  php artisan queue:restart || { echo 'Queue restart failed'; exit 1; }
                  EOF
```
