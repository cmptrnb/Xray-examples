# VLESS-XHTTP-Reality

Update existing software packages:

```
apt update && apt upgrade -y
```

Install Xray-core and geodata with `User=nobody`:

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

Generate a universally unique id:

```
xray uuid
```

Generate a public-private key pair:

```
xray x25519
```

Generate a secret path:

```
< /dev/urandom tr -dc a-z0-9 | head -c${1:-8};echo;
```

Edit server configuration file with `nano` student editor:

```
nano /usr/local/etc/xray/config.json
```

Or with `vi` professional editor:

```
vi /usr/local/etc/xray/config.json
```

Restart `xray` service:

```
systemctl restart xray
```

Check that status of `xray` service is `active (running)`:

```
systemctl status xray
```
