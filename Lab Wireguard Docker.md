

# Wireguard

### Introduction

For this lab.

On your `pi`, stop all unneeded services but `Docker`. Use `netstat -at` to see the list of opened TCP ports on your pi.

To stop a service like `Apache2`

`sudo service apache2 stop`

Make sure `docker-compose` is installed.

## Step 1 — Creating and running the Wireguard container

Open a browser window then navigate to `https://github.com/linuxserver/docker-wireguard`.

Locate the file `README.md` then locate the section `docker-compose.yml`. Copy the content to the clipboard.

Open a ssh session to your raspberry Pi. You then create a `wireguard` folder in the `opt` folder.

```
cd /opt
sudo mkdir wireguard
cd wireguard
```

Now create the `docker-compose.yml` file using `nano`

```bash
sudo nano docker-compose.yml
```

Paste the content of your clipboard (from github) and modify the lines where you see #<====

```bash
---
version: "2.1"
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - SERVERURL=auto #<====
      - SERVERPORT=51820 #optional
      - PEERS=peer1,peer2 #<==== Name of the users
      - PEERDNS=1.1.1.2,1.0.0.2 <====
      - ALLOWEDIPS=0.0.0.0/0 #optional
      - PERSISTENTKEEPALIVE_PEERS= #optional
      - LOG_CONFS=true #optional
    volumes:
      - /opt/wireguard/config:/config #<=====
      - /lib/modules:/lib/modules #optional
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```

Now you create the wireguard container

```bash
docker-compose up -d
```

Check the container is running using

```bash
docker ps -a
```

**Make a screenshot: ContainerRunning.jpg**

## Step 2 — Client configuration

Once the container is created verify the users configuration folders are created.

```bash
:$cd /opt/wireguard/config
:/opt/wireguard/config $ ls
coredns peer_peer1  peer_peer2  server  templates  wg0.conf
```

Verify there is a configuration code png file in the peer_peer1 folder

```bash
:/opt/wireguard/config $ ls peer_peer1
peer_peer1.conf  peer_peer1.png  presharedkey-peer_peer1  privatekey-peer_peer1  publickey-peer_peer1

```

Back on your laptop, copy the png file to your laptop using `scp`. On a Mac to save the file in the `Downloads` folder you use.

```bash
scp pi@Piname.local:/opt/wireguard/config/peer_peer1.png ~/Downloads
```

You will be prompted to enter your pi's password to copy the file.

On your phone App Store, locate and download the wireguard app.

Start the wireguard application then create a new tunnel and use the option `Create from QR code` to add the configuration from `peer_peer1.png` . Use your phone camera to scan the code and create the configuration file.

To start the VPN, use the slider next to the name of your configuration. Then verify you are connected to your Wireguard container. Take a screenshot of your phone screen. **PhoneConnectedVPN.jpg**

## Step 3 — Adding users

To add users to your VPN configuration, follow these steps.

Connect to your Pi then open the docker-compose.yml file

```bash
cd /opt/wireguard
sudo nano docker-compose.yml
```

Locate the Peers section then add the name of the users you want to add. For instance.

```bash
- PEERS=peer1,peer2,jerome
```

Save the file, exit and recreate the container

```bash
docker-compose up -d --force-recreate
```

```bash
cd /opt/wireguad/config
ls
```

**Make a screenshot: NewUsers.jpg**

Upload all screenshots to Github.
