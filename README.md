# EasyArr: HTPC with Docker

> Tested on Debian 10+ (Buster)

> Plex Media Server, Sonarr, Radarr, Bazarr, Prowlarr, qBittorrent, SWAG and WatchTower

Alternatives

- Plex Media Server to Jellyfin: I have a lifetime license for Plex, and I still think it's the best and most complete solution, if you want to install both it won't be difficult to adjust it in Docker.
- qBittorrent to Deluge, Transmission, rTorrent: qBittorrent has the largest number of options with no need for addons, its configuration is fast and has a good performance.

## Security

Create the new public and private keys, then rename and copy and delete the public key:

<pre>
ssh-keygen
cd ~/.ssh/
cp id_rsa.pub authorized_keys
rm -rf id_rsa.pub
</pre>
Copy the key `cat id_rsa` to your computer and then delete`rm -rf id_rsa`. Only with this key will you have access to the server.

`sudo nano /etc/ssh/sshd_config`

<pre>
PermitRootLogin no
PasswordAuthentication no
</pre>

**Ready!** Once the server restarts you will only have access with the private key.

## Update, upgrade and basic packages

Before you start working, update the packages to the latest versions.

<pre>
sudo apt-get update; sudo apt-get upgrade -y; sudo apt-get autoremove

sudo apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common htop ncdu nano sl -y

sudo timedatectl set-timezone Etc/GMT
</pre>

## Swap

This is an optional configuration. I have an environment on a VPS with 1 vCPU and 1GB of ram and an NVME disk, a swap can be important for some tasks.

<pre>
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
</pre>

`echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab`

<pre>
sudo sysctl vm.swappiness=10
sudo sysctl vm.vfs_cache_pressure=50
</pre>

<pre>
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf
</pre>

## Install Docker:

```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"

sudo apt-get update; sudo apt-get -y install docker-ce docker-ce-cli docker-compose -y
```
`sudo systemctl status docker`

You should see a green status. ;)

## Mount disks

If you're starting now, you probably have a disk with a large amount of space, after formatting this process is easier to mount.

Each disk has its own *UUID*.
You can find out the *UUID* of your disk by running `sudo blkid`, the result will be like this:
<pre>
/dev/sda1: UUID="648c6531-4d62-4d3c-a0d9-074f651cdbfb" TYPE="ext4" PARTUUID="6e301172-01"
/dev/sda5: UUID="5e1b25a1-498b-4bce-af47-d30aa9ea31b5" TYPE="swap" PARTUUID="6e301172-05"
/dev/sdb1: LABEL="HD" UUID="<b>CA8261F08261E205</b>" TYPE="ntfs" PARTLABEL="HD" PARTUUID="deb852d4-a496-4b8c-9065-bec8d9c5d7e4"
</pre>

`sudo nano /etc/fstab`
<pre>
UUID=<b>UUID</b> /storage auto defaults 0 0
</pre>

You also need an empty folder where it will be mounted:

<pre>
sudo mkdir -p /storage
sudo chown -R $(whoami). /storage
</pre>

Run `sudo mount -a`. If you can access your partitions and no errors appear then everything is fine.

<pre>
mkdir -p /storage/{tv-shows,movies,downloads}
</pre>

## Install containers
> Plex Media Server, Sonarr, Radarr, Bazarr, Prowlarr, qBittorrent, SWAG and WatchTower

<pre>id</pre>
Get response and change PUID and PGID in *.env*

`nano .env`

Values with # are not required, but are set this way by default.

<pre>
#STORAGE=/storage
#DOCKER=/opt

#PUID=<b>ID</b>
#PGID=<b>ID</b>
#TZ=<b>Etc/GMT</b>

#DNSPLUGIN=<b>cloudflare</b>
#VALIDATION=<b>dns</b>
#SUBDOMAINS=<b>plex,qbittorrent,radarr,sonarr,bazarr,prowlarr</b>

DOMAIN=<b>DOMAIN.XYZ</b>
PLEX_CLAIM=<b>CLAIM_CODE</b>
</pre>

> You can obtain a claim token from https://plex.tv/claim and input here. Keep in mind that the claim tokens expire within 4 minutes.

> SWAG allows the configuration of several [DNS platforms](https://docs.linuxserver.io/general/swag#create-container-via-dns-validation-with-a-wildcard-cert), but Cloudflare is one of the most practical and secure solutions.

> If you need a cheap domain, use [Namecheap Beast Mode](https://www.namecheap.com/domains/registration/results/?type=beast&domain=)

<pre>
rm docker-compose.yml; curl -o docker-compose.yml -L https://raw.githubusercontent.com/altendorfme/easyarr/main/docker-compose.yml

sudo docker-compose up -d --remove-orphans
</pre>

Wait for the complete installation of all applications, it may take several minutes.

## Configuring SWAG

Go to [Cloudflare Dashboard](https://dash.cloudflare.com/) > My Profile > API Tokens

Click on <b>Create Tokens</b>, use template <b>Edit zone DNS</b>.

Add new Permissions with <b>Zone > DNS > Read</b>.

In Zone Resources select <b>Your domain</b>.

Click on <b>Continue the summary</b>, then <b>Create Token</b>

<pre>
echo "dns_cloudflare_api_token = <b>TOKEN</b>" > /opt/swag/dns-conf/cloudflare.ini
</pre>

<pre>
sudo cp /opt/swag/nginx/proxy-confs/plex.subdomain.conf.sample /opt/swag/nginx/proxy-confs/plex.subdomain.conf 
sudo cp /opt/swag/nginx/proxy-confs/qbittorrent.subdomain.conf.sample /opt/swag/nginx/proxy-confs/qbittorrent.subdomain.conf 
sudo cp /opt/swag/nginx/proxy-confs/sonarr.subdomain.conf.sample /opt/swag/nginx/proxy-confs/sonarr.subdomain.conf 
sudo cp /opt/swag/nginx/proxy-confs/radarr.subdomain.conf.sample /opt/swag/nginx/proxy-confs/radarr.subdomain.conf 
sudo cp /opt/swag/nginx/proxy-confs/bazarr.subdomain.conf.sample /opt/swag/nginx/proxy-confs/bazarr.subdomain.conf 
sudo cp /opt/swag/nginx/proxy-confs/prowlarr.subdomain.conf.sample /opt/swag/nginx/proxy-confs/prowlarr.subdomain.conf 
</pre>

`sudo docker restart swag`

## Local web UI ports by applications

- Plex: **32400**
- qBittorrent: **8080**
- Sonarr: **8989**
- Radarr: **7878**
- Bazarr: **6767**
- Prowlarr: **9696**

## Public web UI ports by applications

- Plex: plex.**domain.xyz**
- qBittorrent: qbittorrent.**domain.xyz**
- Sonarr: sonarr.**domain.xyz**
- Radarr: radarr.**domain.xyz**
- Bazarr: bazarr.**domain.xyz**
- Prowlarr: prowlarr.**domain.xyz**

## Path configuration

### Sonarr

<pre>
Settings > Media Management > Importing
- Check <b>Use Hardlinks when trying to copy files from torrents that are still being seeded</b>
</pre>

<pre>
Settings > Media Management > Root Folders
- <b>/storage/tv-shows/</b>
</pre>

### Radarr

<pre>
Settings > Media Management > Importing
- Check <b>Use Hardlinks when trying to copy files from torrents that are still being seeded</b>
</pre>

<pre>
Settings > Media Management > Root Folders
- <b>/storage/movies/</b>
</pre>

### qBittorrent

<pre>
Options > Downloads
- Default Save Path: <b>/storage/downloads</b>
</pre>

### Plex

<pre>
Settings > Network > Custom server access URLs: <b>https://plex.domain.xyz</b>

Settings > Remote Access: <b>Disable Remote Access</b>
</pre>

# References and credits

- https://plex.tv
- https://github.com/Sonarr/Sonarr
- https://github.com/Radarr/Radarr
- https://github.com/morpheus65535/bazarr
- https://github.com/Prowlarr/Prowlarr
- https://github.com/containrrr/watchtower
- https://github.com/ngosang/trackerslist
- https://github.com/linuxserver/docker-swag
- https://www.qbittorrent.org
- https://www.linuxserver.io
- https://trash-guides.info
- https://github.com/chaifeng/ufw-docker