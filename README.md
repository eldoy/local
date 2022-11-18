# Local

Localhost web development with NGINX.

This guide will help you run web applications using a custom domain on your local computer for development.

_Only MacOS is documented at the moment._

### Clone this repo

Start by cloning this repo in your `~/src` directory:

```
git clone https://github.com/eldoy/local.git
```

As an example, we are adding a web application called "Firmalisten" which is running on localhost port `5843`.

#### Stop Apache web server

MacOS has Apache web server pre-installed. Make sure it's not running by running these commands:

```
sudo apachectl stop
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

#### Install NGINX

```
# Install with homebrew
brew install nginx

# Start NGINX server, also on boot
sudo cp /usr/local/opt/nginx/homebrew.mxcl.nginx.plist /Library/LaunchDaemons
sudo launchctl load -w /Library/LaunchDaemons/homebrew.mxcl.nginx.plist

# Restart NGINX server
sudo launchctl kickstart -k system/homebrew.mxcl.nginx

# Stop NGINX server
sudo launchctl unload /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```

Go to [http://localhost](http://localhost) and verify that it works. You should see the text "It works!" in your browser.

#### Add your domain

Add the following line to the `/etc/hosts` file:

```
127.0.0.1 firmalisten.test
```

This will point the URL `firmalisten.test` to localhost.

Your might want to flush the DNS if it's not working:

```
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

#### Configure NGINX

Open the NGINX config file:
```
sudo vim /usr/local/etc/nginx/nginx.conf
```

At the end of the file, add this line to load your custom Virtual Host configurations:
```
# Change the path to your installation
include /Users/vidar/src/local/sites/*.conf;
```

### Add your application

Create a new file in `/Users/vidar/src/local/sites` called `firmalisten.conf`:

```
upstream app {
  server 127.0.0.1:5843 max_fails=0;
  keepalive 2;
}

server {
  listen 80;
  server_name firmalisten.test;

  location / {
    proxy_pass http://app/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_buffering off;
    proxy_request_buffering off;
    proxy_redirect off;
    proxy_cache off;
  }
}
```

This adds a reverse proxy from firmalisten.test to the web application with support for web sockets.

### Reload configuration

From the root directory of this repository, restart NGINX:
```
npm run restart
```

Test your NGINX config in case of problems:
```
npm run test
```

### Commands

This package includes the following convenience scripts defined in `package.json`:

```
# Start NGINX
npm run start

# Restart NGINX
npm run restart

# Stop NGINX
npm run stop

# Test NGINX config
npm run test
```

### Start development

After starting up your web application, go to [http://firmalisten.test](http://firmalisten.test) in your browser, and everything should be working.

Done!
