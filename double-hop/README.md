# How to build a double-hop VPN with WireGuard and Xray VLESS Reality

In this scenario:

* The user connects to VPS 1 with WireGuard
* VPS 1 passes the user's traffic to VPS 2 though an Xray VLESS Reality tunnel
* VPS 2 then transmits the user's requests out to the World Wide Web

## Step 1. Configure VPS 2

Start by downloading the WireGuard install script from https://github.com/angristan/wireguard-install:

```
curl -L https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh -O
```

Make the script executable:

```
chmod +x wireguard-install.sh
```

Execute the script:

```
./wireguard-install.sh
```

Explicitly specify the listening port as `51820/udp`.

Toward the end of the script's run, you will be prompted to specify a client name. We will use `mypc` as our example.

We do not need to open the firewall for WireGuard, since it will be behind Xray. Therefore edit the generated server configuration file:

```
vi /etc/wireguard/wg0.conf
```

Delete the firewall opening lines:

```
PostUp = iptables -I INPUT -p udp --dport 51820 -j ACCEPT
```

And:

```
PostDown = iptables -D INPUT -p udp --dport 51820 -j ACCEPT
```

Write the file to disk, and quit the editor.

The easiest way to implement these changes right now is to reboot:

```
reboot
```

After a few moments, SSH back into VPS2.

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

Open port `tcp/443` in the firewall and/or security groups for VPS 2.

You now have Xray listening on port `tcp/443`, and WireGuard listening on localhost port `udp/51820` (which need not be open in this server’s firewall).


## Step 2. Configure VPS 1

Install Xray from https://github.com/XTLS/Xray-install:

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --beta -u root
```

For configuring Xray, you can use the `relay.json` from this repository as your starting point:

```
curl -L https://raw.githubusercontent.com/cmptrnb/Xray-examples/refs/heads/main/double-hop/relay.json -o /usr/local/etc/xray/config.json
```

Note that the Xray client accepts `dokodemo-door` input on port `udp/51820`.

Edit the Xray relay configuration file:

```
vi /usr/local/etc/xray/config.json
```

At a minimum, replace the UUID, the public key, and the IP address of VPS 2 in the example with your own values.

Write the file to disk, and quit the editor.

Restart the Xray systemd service with this configuration file:

```
systemctl restart xray
```

Open port `udp/51820` in this VPS's firewall and/or security groups.

This server is now listening for public input on `udp/51820`, and whatever it gets (`dokodemo-door`) will be sent to the server VPS 2.

## Step 3. Configure user PC

Install WireGuard on the client as per https://www.wireguard.com/install. For example, for a Debian Linux client:

```
sudo apt install -y wireguard
```

Securely download the generated client configuration file from VPS2:

```
scp root@VPS2.SERVER.IP.ADDRESS:/root/wg0-client-mypc.conf .
```

Edit the apparent destination to be your relay server (VPS 1) IP address, instead of your final server (VPS 2) IP address:

```
Endpoint = VPS1.SERVER.IP.ADDRESS:51820
```

Write the file to disk, and quit the editor.

Copy the WireGuard configuration file into place:

```
cp wg0-client-mypc.conf /etc/wireguard/wg0.conf
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

