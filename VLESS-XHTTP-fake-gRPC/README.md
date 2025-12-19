# VLESS XHTTP fake gRPC

gRPC (Google Remote Procedure Call) is an open-source framework that allows applications to communicate across different languages and platforms. It uses HTTP/2 for transport.

The settings for fake gRPC come from https://github.com/pomidorka1515/xray-examples.

You will need your own domain name for this scenario.


## Step 1. Open server firewall

Open ports `tcp/80` and `tcp/443` in your server firewall and/or VPS security groups. 

Since the ACME certificate script needs to use port `tcp/80`, that port needs to be open, but nothing else must use it. 

Port `tcp/8443`, where Xray will listen on localhost, does not need to be open for public input.


## Step 2. Get SSL certificate and key

SSH into your server as root, replacing `your.server.hostname` in the command below with your actual server hostname:

```
ssh root@your.server.hostname
```

Get your server packages up to date:

```
apt update && apt upgrade -y
```

Install the prerequisite for the ACME certificate script:

```
apt install -y socat
```

Install the ACME certificate script:

```
curl https://get.acme.sh | sh
```

Set a shorter alias for the ACME shell script:

```
alias acme.sh=~/.acme.sh/acme.sh
```

Set up ACME shell script auto-update:

```
acme.sh --upgrade --auto-upgrade
```

Change the default Certificate Authority to Let's Encrypt:

```
acme.sh --set-default-ca --server letsencrypt
```

Apply for an Elliptic Curve Cryptography certificate for `your.server.hostname` in standalone mode. ec-256 means prime256v1 also known as ECDSA P-256. Replace `your.server.hostname` in the example command with your actual server hostname.

```
acme.sh --issue -d your.server.hostname --standalone --keylength ec-256
```

Install the `your.server.hostname` certificate to the /etc/ssl/private directory. Replace `your.server.hostname` in the example command with your actual server hostname.

```
acme.sh --install-cert -d your.server.hostname --ecc --fullchain-file /etc/ssl/private/fullchain.cer --key-file /etc/ssl/private/private.key
```

Set the owner and group to allow access to the private key by non-root applications:

```
chown -R nobody:nogroup /etc/ssl/private
```

Set everything up for certificate renewal. Replace `your.server.hostname` in the example command with your actual server hostname.

```
acme.sh --renew -d your.server.hostname --force --ecc
```


## Step 3. Configure Nginx as a reverse proxy

Install Nginx:

```
apt install -y nginx
```

Download the model Nginx configuration for this scenario:

```
curl -L https://raw.githubusercontent.com/cmptrnb/Xray-examples/refs/heads/main/VLESS-XHTTP-gRPC/nginx.conf -o /etc/nginx/nginx.conf
```

Edit your Nginx configuration:

```
vi /etc/nginx/nginx.conf
```

Change `your.server.hostname` to be your actual hostname.

Change the line that sets the `$website` variable to refer to your preferred camouflage content.

Write the file to disk, and quit the editor.

Check your Nginx configuration file:

```
nginx -t
```

Restart Nginx with your new configuration:

```
systemctl restart nginx
```

Check the status of the Nginx service:

```
systemctl status nginx
```


## Step 4. Install and configure Xray

Install Xray:

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

Download the model server configuration:

```
curl -L https://raw.githubusercontent.com/cmptrnb/Xray-examples/refs/heads/main/VLESS-XHTTP-gRPC/server.json -o /usr/local/etc/xray/config.json
```

The secret path is given in the examples as `z1gll8p35ybm75r6`. In the real world, you should generate your own pseudo-random, 16-character secret path by issuing the command:

```
< /dev/urandom tr -dc a-z0-9 | head -c${1:-16};echo;
```

The UUID is given in the examples as `6cd3b55f-afb6-412a-8f1c-d0260c409aa0`. In the real world, you should generate your own UUID:

```
xray uuid
```

Edit your server configuration:

```
vi /usr/local/etc/xray/config.json
```

Change the id (UUID), the secret path, and `your.server.hostname`.

Write the file to disk, and quit the editor.

Restart Xray with your modified configuration file:

```
systemctl restart xray
```

Check the status of the Xray service:

```
systemctl status xray
```

Exit your SSH session with the server:

```
exit
```


## Step 5. Set up client

There are many Xray clients. If you are not familiar with them, please refer to the list of clients in the [Xray-core README](https://github.com/XTLS/Xray-core).

We will use Linux in this example, but an equivalent process will apply on other platforms.

Download, configure, and run the Xray-core client as follows:

1. Download the most recent Linux 64-bit executable for Xray-core from https://github.com/XTLS/Xray-core/releases. 
2. Unzip the downloaded file Xray-linux-64.zip.
3. Open a browser at https://github.com/cmptrnb/Xray-examples.
4. Select the folder `VLESS-XHTTP-gRPC`.
5. Select the file `client.json`.
6. Click the **Raw** button.
7. Copy the contents to your PC clipboard.
8. Open a text editor.
9. Paste the contents of `client.json` from your PC clipboard into the text editor.
10. Modify the values as needed (UUID, secret path, and `your.server.hostname` in 3 places).
11. Save the file with the name `config.json` in the same directory as `xray`.
12. Open a terminal window and run the Xray Linux client by issuing the command:

```
./xray -c config.json
```

Leave the terminal window open with Xray running in it.

Set up Firefox proxying like this:

1. From the Firefox hamburger menu, open **Settings**.
2. On the **General** page, under **Network Settings**, click the **Settings** button.
3. Select **Manual proxy configuration**.
4. Set the SOCKS Host to `127.0.0.1`, and set the Port to `10808`.
5. Select **SOCKS v5**.
6. Click **OK**.

Now test your entire configuration. Open a browser and visiting a site such as https://ipchicken.com.
