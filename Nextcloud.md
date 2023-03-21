

# Nextcloud

### Objectives

In this lab you:

- set up your Pi as a Nextcloud server.

**1.** The first thing we need to do is ensure our Raspberry Pi is using the latest available packages.

We can do that by running the following two commands.

```
sudo apt update
sudo apt upgrade
```



### Step 1 — Creating and running the Nextcloud container

Open a browser window then navigate to `https://hub.docker.com/r/linuxserver/nextcloud`.

Locate the file `README.md` then locate the section `docker-compose.yml`. Copy the content to the clipboard.

Open a ssh session to your raspberry Pi. You then create a `wireguard` folder in the `opt` folder.

```
cd /opt
sudo mkdir nextcloud
cd nextcloud
sudo mkdir data
```

Now create the `docker-compose.yml` file using `nano`

```bash
sudo nano docker-compose.yml
```

Paste the content of your clipboard (from github) and modify the lines where you see #<====

```
---
version: "2.1"
services:
  nextcloud:
    image: lscr.io/linuxserver/nextcloud:latest
    container_name: nextcloud
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /opt/nextcloud:/config   					#<=========
      - /opt/nextcloud/data:/data					#<=========
    ports:
      - 3443:443
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

### Step 2 — Starting nextcloud

Once the container is started, you can access nextcloud from a web browser using the following URL: `https://IpOfYourPi:3443` when IpOfYourPi is the IP address of your Pi.

**Make a screenschot nextcloud1.jpg**

### Step 3 — Starting nextcloud with Docker/Apache2

Make sure you stop the previous container. 

```
docker ps -a
docker stop cef391be19c4 #if cef391be19c4 is the container id
docker container prune
```

Make sure Apache 2 is installed and started on your Pi. Then issue the command:

```console
docker run -d -p 8080:80 nextcloud
```

Once the container is started, you can access nextcloud from a web browser using the following URL: `http://IpOfYourPi:8080` when IpOfYourPi is the IP address of your Pi.

**Make a screenschot nextcloud2.jpg**

Upload the screenshots to github

