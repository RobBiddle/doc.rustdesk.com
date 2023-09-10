---
title: Configure Relay Servers
weight: 17
---

# RustDesk Pro - Install Additional Relay Servers with Geo Location using docker

You can have several relay servers running across the globe and leverage GeoLocation to use the closest relay server, giving you a faster experience when connecting to remote computers.

> [!IMPORTANT]
> You will need the private key pair **id_ed25519** and **id_ed25519.pub** 

If docker is already installed, connect to your server via SSH and create a volume for HBBR
```
# docker volume create hbbr
```
The volume hbbr should be located in /var/lib/docker/volumes/hbbr/_data

Copy the private key pair to the volume location, in this case we will use SCP to copy the files. 

The command syntax is: scp <path/filename> username@server:<file path destination>

```
# scp id_ed25519 root@100.100.100.100:/var/lib/docker/volumes/hbbr/_data
# scp id_ed25519.pub root@100.100.100.100:/var/lib/docker/volumes/hbbr/_data
```


Deploy the HBBR container using the volume previously created. This volume has the private key pair needed to run your private relay.

```
# sudo docker run --name hbbr -v hbbr:/root -td --net=host rustdesk/rustdesk-server hbbr -k _
```

check the running logs to verify that hbbr is running using your key pair

`# docker logs hbbr`

INFO [src/common.rs:121] **Private key comes from id_ed25519**
NFO [src/relay_server.rs:581] Key: XXXXXXXXXXXXXXXXXXXXX
INFO [src/relay_server.rs:60] #blacklist(blacklist.txt): 0
INFO [src/relay_server.rs:75] #blocklist(blocklist.txt): 0
INFO [src/relay_server.rs:81] Listening on tcp :21117


Depending on your OS, you might want to block/allow IPs using a firewall.

In our case, running ubuntu we want to allow any tcp connections, to ports 21117 and 21119

```
# sudo ufw allow proto tcp from any to any port 21117,21119
```

**enable the firewall**
```
# sudo ufw enable
```

**check the status**
```
# ufw status

Status: active

To                         		 	 Action          From
--                         ------      ----
21117,21119/tcp            	ALLOW       Anywhere
21117,21119/tcp (v6)     	ALLOW       Anywhere (v6) 
```


## Configure HBBS Pro for Geo Location

### Register and Download the GeoLite2 City database file


To use geo location, HBBS needs access to the MaxMind GeoLite2 City database. The database is free and you can register to download the file and get an API key.

Start by creating an account (if you don’t have one) by going to the website https://www.maxmind.com/en/account/login
Go to Download Databases and download the GeoLite2 City, choose the gzip file and you should have the mmdb when decompressing it.

<img width="1119" alt="image" src="https://github.com/rustdesk/doc.rustdesk.com/assets/642149/e14318fb-ec52-463c-af77-d08c9479c1b5">


If you installed RustDesk Pro using the installation script on a Linux machine, the mmdb file needs to be moved to **/var/lib/rusted-server/**

For docker installations the file should be in the volume you mapped when deploying the container mapped to \/root

#### Get an API key to automate the process - Linux servers
You need to update this file regularly and we can use a cronjob to do that. You will need an API key to access the download link which is free.

Go to **Manage License Keys**
<img width="329" alt="image" src="https://github.com/rustdesk/doc.rustdesk.com/assets/642149/632aeb33-4f5d-4a31-9010-38e01c22d3c9">



Generate a new license key and save the key
<img width="1064" alt="image" src="https://github.com/rustdesk/doc.rustdesk.com/assets/642149/3e178174-5fbf-46b7-a335-01f77125dfad">


You can automate the download process in a few ways (https://dev.maxmind.com/geoip/updating-databases) but you add the following command to your crontab replacing {Your Access Key} with the API key you got from the previous step.

```
/usr/bin/curl -L --silent 'https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-City&license_key={Your Access Key}&suffix=tar.gz' | /bin/tar -C '/var/lib/rustdesk-server/‘’ -xvz --keep-newer-files --strip-components=1 --wildcards '*GeoLite2-City.mmdb'
```


## Change settings in RustDesk Pro Web Console

Add your relay server IP addresses to the the Relay Server List, using just the IP address. **Do not add the port**

<img width="778" alt="image" src="https://github.com/rustdesk/doc.rustdesk.com/assets/642149/c4452ba4-5e1d-437a-ae1d-fc0070bfa26c">


Add a Geo Override but adding the server IP address and the coordinates where the server is located.

<img width="502" alt="image" src="https://github.com/rustdesk/doc.rustdesk.com/assets/642149/41c558e3-423b-4296-90d3-cb0769f4a369">


Click Reload Geo and your list should look similar to this

<img width="1568" alt="image" src="https://github.com/rustdesk/doc.rustdesk.com/assets/642149/5a0d39a9-4fec-46b4-a7a2-7ed38b6baeb7">


To confirm the results, check your HHBS logs when clicking Reload Geo, you should see a message showing the relay server IP addresses and their coordinates