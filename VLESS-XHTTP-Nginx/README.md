# VLESS XHTTP with Nginx in front

In this arrangement, the client talks to Nginx on the server. Nginx handles TLS. If the client has specified the correct secret path, the request is passed to Xray.

You must have your own domain name to construct a server with this configuration.



## Step 1. Install Nginx

SSH into your server:

```
ssh root@your.server.hostname
```

Update your existing software packages:

```
apt update && apt upgrade -y
```

Install Nginx of at least version 1.25.1, since this version is when `http2 on` was introduced:

```
apt install -y nginx
```

Note that Debian 13 includes Nginx version 1.26.3.

Edit the Nginx site configuration file:

```
vi /etc/nginx/sites-available/default
```

Specify your server's domain name:

```
        server_name your.server.hostname;
```

At this stage, your configuration file should look something like this:

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        server_name your.server.hostname;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

Write the file to disk, and quit the editor.

Restart Nginx with the site configuration file that now specifies your server name:

```
systemctl restart nginx
```



## Step 2. Add web site content

You will add some sample content from https://github.com/PavelDoGreat/WebGL-Fluid-Simulation. Download the zipped content from that repository:

```
curl -L https://github.com/PavelDoGreat/WebGL-Fluid-Simulation/archive/refs/heads/master.zip -O
```

Unzip the site content:

```
apt install -y unzip
```

```
unzip master.zip
```

Copy the content into place:

```
cp -r WebGL-Fluid-Simulation-master/* /var/www/html/
```


## Step 3. Add SSL certificate and key

Install snap:

```
apt install -y snapd
```

Install the EFF certbot script:

```
snap install --classic certbot
```

Put a link to certbot in your path:

```
ln -s /snap/bin/certbot /usr/bin/certbot
```

Invoke certbot for an Nginx server:

```
certbot --nginx
```

You will be prompted to:

* Enter your email address, or leave it blank
* Agree to the terms of service
* Select your hostname by number, or press Enter to select all 

Once the SSL certificate is issued and installed, set everything up for periodic renewal of your SSL certificate and key:

```
certbot renew --dry-run
```


## Step 4. Generate secret path

In the example server and client JSON files, you will see a secret path of `vtcecvvc`.

You can generate your own pseudo-random, 8-character path name with the command:

```
< /dev/urandom tr -dc a-z0-9 | head -c${1:-8};echo;
```


## Step 5. Modify Nginx site configuration

Edit the Nginx site configuration file:

```
vi /etc/nginx/sites-available/default
```

Add HTTP/2 support:

```
        http2 on;
```

(The `http2 on` command was added in Nginx 1.25.1.)

Add proxy pass to localhost port `1234`, which is where Xray will be listening:

```
        location /vtcecvvc {
                proxy_pass http://127.0.0.1:1234;
                proxy_http_version 1.1;
                proxy_redirect off;
        }
```

Replace the secret path name in the example (`vtcecvvc`) with your actual secret path name.

At this stage, your Nginx site configuration file will look something like the `default` file in this repository.

Write the Nginx site configuration file to disk, and quit the editor.

Restart Nginx for this change:

```
systemctl restart nginx
```


## Step 6. Install Xray

Install xray-core latest stable version:

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```


## Step 7. Generate UUID

The example uses a UUID of `0f8f47c3-bfe3-4757-8bcb-3dfcc72c9e9f`. You can generate your own universally unique id by issuing the command:

```
xray uuid
```


## Step 8. Configure server

Edit the Xray server configuration file:

```
vi /usr/local/etc/xray/config.json
```

You can use the `server.json` from this repository as your starting point. At a minimum, replace the UUID and the secret path name in the example with your own values.

Once you've done making changes, write the Xray server configuration file to disk, and quit the editor.


## Step 10. Restart Xray on server

```
systemctl restart xray
```

Xray is now listening on port `1234` for input from localhost (`127.0.0.1`) only.

End your SSH session with the server:

```
exit
```

## Step 11. Download client

Open a browser and navigate to https://github.com/XTLS/Xray-core/releases.

Download the most recent executable for your client platform, e.g. `Xray-linux-64.zip`.

Unzip the zip file.

In a terminal window, change into the unzipped directory. For example:

```
cd Downloads\Xray-linux-64
```


## Step 12. Configure client

Edit the Xray client configuration file:

```
vi config.json
```

You can use the `client.json` from this repository as your starting point. At a minimum, replace the UUID, the secret path name, and `your.server.hostname` in the example with your own values.

Write the Xray client configuration file to disk, and quit the editor.


## Step 13. Run client

Still positioned in the Xray client directory, run the Xray client:

```
./xray -c config.json
```

On Windows, the command would be:

```
xray.exe -c config.json
```

Leave the terminal window open with Xray running in it.


## Step 14. Configure Firefox

Configure Firefox (Settings &gt; General &gt; Network Settings) to use the SOCKS5 proxy server on `127.0.0.1` port `10808`.












