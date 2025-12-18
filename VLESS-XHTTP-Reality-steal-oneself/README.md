# VLESS XHTTP Reality "steal oneself"

In this configuration, the Xray client sends XHTTP requests to your server with your own domain name ("steal oneself"). These requests specify a secret path.

The Xray server listens for requests on port `tcp/443`. 

If the secret path is incorrect, Xray sends the requests to Nginx listening on localhost port `tcp/8001`. Nginx acts as a reverse proxy to serve up your camouflage content.

You will need your own domain name for this scenario. The hostname of your VPS is given in the examples as `your.server.hostname`.

The configuration files in this folder were forked from https://github.com/chika0801/Xray-examples.


## Step 1. Open server firewall

Open ports `tcp/80` and `tcp/443` in your server firewall and/or VPS security groups. 

Since the ACME certificate script needs to use port `tcp/80`, that port needs to be open, but nothing else must use it. 

Port `tcp/8001`, where the Nginx reverse proxy will listen on localhost, need not be open for public input.


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

Apply for an Elliptic Curve Cryptography certificate for your.server.hostname in standalone mode. ec-256 means prime256v1 also known as ECDSA P-256. Replace `your.server.hostname` in the example command with your actual server hostname.

```
acme.sh --issue -d your.server.hostname --standalone --keylength ec-256
```

Install the your.server.hostname certificate to the /etc/ssl/private directory. Replace `your.server.hostname` in the example command with your actual server hostname.

```
acme.sh --install-cert -d your.server.hostname --ecc --fullchain-file /etc/ssl/private/fullchain.cer --key-file /etc/ssl/private/private.key
```

Set the owner and group to support non-root applications:

```
chown -R nobody:nogroup /etc/ssl/private
```

Set everything up for certificate renewal. Replace `your.server.hostname` in the example command with your actual server hostname.

```
acme.sh --renew -d your.server.hostname --force --ecc
```


## 3. Configure Nginx as a reverse proxy

Install Nginx:

```
apt install -y nginx
```

Download the model Nginx configuration for this scenario:

```
curl -L https://raw.githubusercontent.com/cmptrnb/Xray-examples/refs/heads/main/VLESS-XHTTP-Reality-steal-oneself/nginx.conf -o /etc/nginx/nginx.conf
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
curl -L https://raw.githubusercontent.com/cmptrnb/Xray-examples/refs/heads/main/VLESS-XHTTP-Reality-steal-oneself/server.json -o /usr/local/etc/xray/config.json
```

The secret path is given in the examples as `c6gim9rw`. You can generate your own pseudo-random, 8-character secret path with the command:

```
< /dev/urandom tr -dc a-z0-9 | head -c${1:-8};echo;
```

The UUID is given in the examples as `a4b77f56-1fe6-485e-9b48-48bb198ce784`. Generate your own UUID:

```
xray uuid
```

The private and public key (`Password`) are given in the examples as:

```
PrivateKey: cCxc5EJIDFlqlp5uFXLIo_OMTXzwmMlztmitB2CIw3s
Password: VqCnBCOjZ2xvj0fquZpCQEyzpZtMhr4-JvkNK23jd3E
```

Generate your own public and private key:
```
xray x25519
```

Edit your server configuration:

```
vi /usr/local/etc/xray/config.json
```

Change the id (UUID), the secret path, `your.server.hostname`, and the private key.

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

There are many Xray clients.

If you are not familiar with them, see the [Xray-core README](https://github.com/XTLS/Xray-core) for lists of GUI and non-GUI clients. 

We'll use Windows in this example.

Download, configure, and run the Xray-core Windows client as follows:

1. Download the most recent Windows executable for Xray-core from https://github.com/XTLS/Xray-core/releases. 
2. Unzip the downloaded zip file Xray-windows-64.zip.
3. Open a browser at https://github.com/cmptrnb/Xray-examples.
4. Select the folder `VLESS-XHTTP-Reality-steal-oneself`.
5. Select the file `client.json`.
6. Click the **Raw** button.
7. Copy the contents to your Windows clipboard.
8. Open Windows Notepad.
9. Paste the contents of `client.json` from your clipboard into Windows Notepad.
10. Modify the values as needed (UUID, secret path, public key, and `your.server.hostname` in 2 places).
11. Save the file with the name `config.json` in the same directory as `xray.exe`.
12. Open a Windows Command Prompt and run the Windows client by issuing the command:

```
xray.exe -c config.json
```

Leave the Command Prompt window open with Xray running in it.

Set up system-wide proxying on Windows like this:

1. Open the Windows **Settings** app. 
2. Select **Network & internet**.
3. Select **Proxy**.
4. Under **Manual proxy set up**, on the line marked **Use a proxy server**, select **Set up**.
5. Turn on the proxy, set the Proxy IP address to `127.0.0.1`, and set the Port to `10808`.

Now test your entire configuration. Open a browser and visiting a site such as https://ipchicken.com.
