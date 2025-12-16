# How to build a double-hop VPN with WireGuard and Xray VLESS Reality

In this scenario:

* The user connects to VPS 1 with WireGuard
* VPS 1 passes the user's traffic to VPS 2 though an Xray VLESS Reality tunnel
* VPS 2 then transmits the user's requests out to the World Wide Web

## Step 1. Configure VPS 2

SSH into VPS 2.

Start by downloading and running the WireGuard install script from https://github.com/Nyr/wireguard-install:

```
wget https://git.io/wireguard -O wireguard-install.sh && bash wireguard-install.sh
```

Specify the WireGuard server listening port as `51820`.

You will be prompted to specify a client name. We will use `mypc` as our example.

Check the status of WireGuard with: 

```
systemctl status wg-quick@wg0
```

Install Xray using the script from https://github.com/XTLS/Xray-install:

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

Generate a UUID:

```
xray uuid
```

Generate a public-private key pair:

```
xray x25519
```

For configuring Xray, you can use the server.json from this repository as your starting point:

```
curl -L https://raw.githubusercontent.com/cmptrnb/Xray-examples/refs/heads/main/double-hop/server.json -o /usr/local/etc/xray/config.json
```

Edit the Xray server configuration file:

```
vi /usr/local/etc/xray/config.json
```

At a minimum, replace the UUID and private key in the example with your own values.

Write the file to disk, and quit the editor.

Restart the Xray systemd service with this configuration file:

```
systemctl restart xray
```

Open port `tcp/443` in the VPS 2 firewall and/or security groups.

Exit your SSH session with VPS 2:

```
exit
```


## Step 2. Configure VPS 1

SSH into VPS 1.

Install Xray from https://github.com/XTLS/Xray-install:

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

For configuring Xray, you can use the `relay.json` from this repository as your starting point:

```
curl -L https://raw.githubusercontent.com/cmptrnb/Xray-examples/refs/heads/main/double-hop/relay.json -o /usr/local/etc/xray/config.json
```

Edit the Xray relay configuration file:

```
vi /usr/local/etc/xray/config.json
```

Note that the Xray client accepts `dokodemo-door` input on port `udp/51820`.

At a minimum, replace the UUID, the public key, and the `VPS2.SERVER.IP.ADDRESS` in the example with your own values.

Write the file to disk, and quit the editor.

Restart the Xray systemd service with this configuration file:

```
systemctl restart xray
```

This server is now listening for public input on `udp/51820`, and whatever it receives (`dokodemo-door` inbound) will be output to the server VPS 2.

Open port `udp/51820` in this VPS's firewall and/or security groups.

Exit your SSH session with VPS 1:

```
exit
```

## Step 3. Configure user PC

Install WireGuard on the client as per https://www.wireguard.com/install. For example, for a Debian Linux client:

```
sudo apt install -y wireguard
```

Securely download the generated client configuration file from VPS2:

```
scp root@VPS2.SERVER.IP.ADDRESS:/mypc.conf Downloads
```

Edit the apparent destination to be your relay server (VPS 1) IP address, instead of your final server (VPS 2) IP address:

```
Endpoint = VPS1.SERVER.IP.ADDRESS:51820
```

Write the file to disk, and quit the editor.

Copy the WireGuard configuration file into place:

```
cp Downloads/mypc.conf /etc/wireguard/wg0.conf
```

Connect your client to the relay server (VPS 1), which will in turn connect over Xray to the final server (VPS 2).

```
sudo wg-quick up wg0
```

This brings the tunnel up. Verify the Connection:

```
sudo wg show
```


## Step 4. End-to-end test

Test the connection all the way from the client out to the World Wide Web by visiting a site such as https://iplocation.io.

