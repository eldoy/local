# Local

Localhost web development with Apache.

This guide will help you run web applications using a custom domain on your local computer for development.

_Only MacOS is documented at the moment._

### MacOS

MacOS has Apache built in by default.

#### Start Apache

```
# Start apache
sudo apachectl start
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

#### Configure Apache

Open the Apache config file:
```
sudo vim /etc/apache2/httpd.conf
```

Make sure the following modules are enabled (uncommented):

```
LoadModule proxy_module libexec/apache2/mod_proxy.so
LoadModule proxy_http_module libexec/apache2/mod_proxy_http.so
LoadModule proxy_wstunnel_module libexec/apache2/mod_proxy_wstunnel.so
LoadModule rewrite_module libexec/apache2/mod_rewrite.so
```

At the end of the file, add this line to load your custom Virtual Host configurations:
```
# Change the path to your installation
Include /Users/vidar/src/local/config/*.conf
```

### Add your application

As an example, we are adding a web application called "Firmalisten" which is running on localhost port `5843`.

Create a new file in `/Users/vidar/src/local/config` called `firmalisten.conf`:

```
<VirtualHost *:80>
    ServerName firmalisten.test
    ProxyRequests on
    RequestHeader set X-Forwarded-Proto "http"
    ProxyPass / http://localhost:5843/
    ProxyPassReverse / http://localhost:5843/

    RewriteEngine on
    RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC]
    RewriteCond %{HTTP:CONNECTION} ^Upgrade$ [NC]
    RewriteRule .* ws://localhost:5843%{REQUEST_URI} [P]
</VirtualHost>
```

This adds a reverse proxy from firmalisten.test to the web application with support for web sockets.

### Reload configuration

Restart Apache:
```
sudo apachectl restart
```

Test your Apache config:
```
apachectl configtest
```

### Commands

This package includes the following convenience scripts:

```
# Start Apache
npm run start

# Restart Apache
npm run restart

# Test Apache config
npm run test
```

### Start development

After starting up your web application, go to [http://firmalisten.test](http://firmalisten.test) in your browser, and everything should be working.

Done!
