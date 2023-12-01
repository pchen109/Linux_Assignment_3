## Step 1: Access Your Debian 12 Server
- This `-i` option  in `SSH` command specifies the identity (private key) file to be used for authentication.
1. Copy IP address from DigitalOcean.
2. Enter the command to sign in vis `SSH`.
	1. Replace `<IP address>` with your copied IP address.
``` bash
ssh -i .ssh/do-key root@<IP address>
```


## Step 2: Create a New Regular User with a Defined Login Shell
- This `-m` option in `useradd` command creates the user's home directory.
- The `-s SHELL` in `useradd` command specifies a path to the user's login shell.
1. Add a new regular user.
	1. Replace `<user-name>` with your preferred username.
```bash
useradd -ms /bin/bash <user-name>
```

Congratulation, you have created your own user that has bash as login shell.

## Step 3: Add the New User to the `sudo` group
- This `-a` option in `usermod` command appends user to a group without removing the user from other groups.
	- used only with the `-G` option
- This `-G` option in `usermod` command specifies the group where the user should belong to.
	- If no `-a` option, the user can be removed from other supplementary groups.
1. Add the user to the `sudo` group.
	1. Replace `<user-name>` with your username.
``` bash
usermod -aG sudo <user-name>
```

2. Create a password
	- You will be asked to enter password twice.
	- Note that you cannot see what you type for security purpose.
	- Replace `<user-name>` with your username
```
passwd <user-name>
```

Congratulation, you can now perform administrative tasks with `sudo` before any command as a regular user. You need to enter password when using `sudo` as a regular user. 


## Step 4: Allow your user to log in via SSH
- This `-r` option in `cp` command allows you to copy a directory itself and all files within the directory to another directory.
- This `-R` option for `chown` command change owner for all files within the directory and the directory itself.
- Don't need `sudo` because you are logging in as a `root` user. 
1. Replace all `<user-name>` with your user name in the commands below.
2. Enter the commands.
```bash
cp -r /root/.ssh /home/<user-name>
chown -R <user-name>:<user-name> /home/<user-name>/.ssh
```

Congratulation, you can now log in as a regular user via SSH.

## Step 5: Prevent Root User from Connecting to the Server via SSH
- If you are a regular user, you need `sudo` for every following commands.

1. Open the `sshd_config` file.
```bash
vim /etc/ssh/sshd_config
```

2. Type `/Permit` to find the line `PermitRootLogin yes`
	1. Use `n` to navigate to the next matching one.

3. Change the line from `PermitRootLogin yes` to `PermitRootLogin no`.

4. Press `esc` and `:x` to save changes in and exit the file.

5. Restart the SSH service to for the change to take effect.
```bash
systemctl restart ssh.service
```

6. Enter `exit` in your terminal to exit the server.

7. Attempt to log in as the root user
	1. replace `<IP address>` with your IP address
``` bash
ssh -i .ssh/do-key root@<IP address>
```

You should see: `root@<IP Address>` Permission denied (publickey). 
Congratulation, you have successfully prevented root from logging in via SSH. 


## Step 6: Install `nginx`
1. Log in as a regular user via SSH with the command below
	- replace `<user-name>` and `<IP address>` with yours
``` bash
ssh -i .ssh/do-key <user-name>@<IP address>
```

2. Install `nginx` with this command
	1. press `y` for any prompt
```bash
sudo apt install nginx
```

3. Check status of `nginx` to ensure it's active
``` bash
sudo systemctl status nginx
```

4. Enable `nginx` to start automatically on system boot
```bash
sudo systemctl enable nginx
```

Congratulation, you have successfully installed `nginx` which starts on boot.

## Step 7: Configure `nginx` to Serve a Sample Website
1. Make a `my-site` directory inside `/var/www` directory.
``` bash
sudo mkdir /var/www/my-site
```

2. Create and open `index.html` inside `/var/www/my-site` directory.
```bash
sudo vim /var/www/my-site/index.html
```

3. Past this inside `index.html` file.
```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>
```

4. Press `esc` key and enter `:x` to save and leave `index.html` file

5. Create and open `my-site.conf` file inside `/etc/nginx/sites-available` directory
```bash
sudo vim /etc/nginx/sites-available/my-site.conf
```

6. Paste this inside `my-site.conf` file.
```bash
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##
# Default server configuration
#
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	# SSL configuration
	#
	# listen 443 ssl default_server;
	# listen [::]:443 ssl default_server;
	#
	# Note: You should disable gzip for SSL traffic.
	# See: https://bugs.debian.org/773332
	#
	# Read up on ssl_ciphers to ensure a secure configuration.
	# See: https://bugs.debian.org/765782
	#
	# Self signed certs generated by the ssl-cert package
	# Don't use them in a production server!
	#
	# include snippets/snakeoil.conf;
	root /var/www/my-site/;
	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;
	server_name _;
	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
	# pass PHP scripts to FastCGI server
	#
	#location ~ \.php$ {
	#	include snippets/fastcgi-php.conf;
	#
	#	# With php-fpm (or other unix sockets):
	#	fastcgi_pass unix:/run/php/php7.4-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
	#}
	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	#location ~ /\.ht {
	#	deny all;
	#}
}
# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#	listen 80;
#	listen [::]:80;
#
#	server_name example.com;
#
#	root /var/www/example.com;
#	index index.html;
#
#	location / {
#		try_files $uri $uri/ =404;
#	}
#}
```

7. Remove `default` symbolic link inside `/etc/nginx/sites=enabled` directory.
``` bash
sudo unlink /etc/nginx/sites-enabled/default
```

8. Create a symbolic link of `my-site.conf` inside `/etc/nginx/sites-enabled` directory.
``` bash
sudo ln -s /etc/nginx/sites-available/my-site.conf /etc/nginx/sites-enabled
```

9. Test the syntax of the Nginx configuration.
	- Successful message:
		- nginx: the configuration file `/etc/nginx/nginx.conf` syntax is ok
		- nginx: configuration file `/etc/nginx/nginx.conf` test is successful
```bash
sudo nginx -t
```

10. Restart the SSH service to for the change to take effect.
```bash
sudo systemctl restart nginx
```

11. Use `curl` to access the changed HTML
		1. You should see a HTML with "Hello, World" in `<h1>` tag
		2. replace `<IP-address>` with yours.

```bash
curl <IP-address>
```

Congratulation, you have successfully configured `nginx` to serve a sample website containing "Hello, World".

