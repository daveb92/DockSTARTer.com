---
layout: default
---

## General Setup
Out of the box, the LetsEncrypt container created by [linuxserver.io](https://www.linuxserver.io/) performs reverse proxy functions using [NGINX](https://www.nginx.com/) and automatic https encrypted connections using certificates provided by [LetsEncrypt](https://letsencrypt.org/). More on this container can be found [here](https://hub.docker.com/r/linuxserver/letsencrypt/).

To configure your reverse proxy, consider if you want to use subfolders (ie. domain.com/portainer) or subdomains (ie. portainer.domain.com). Subdomains will take more configuration, as DNS entries and certificate subject alternate names are required.

The first thing to setup is your domain and email settings in `.docker/compose/.env` under LETSENCRYPT. Set the `LETSENCRYPT_EMAIL` and `LETSENCRYPT_URL`. If using subdomains ensure to add each subdomain to `LETSENCRYPT_SUBDOMAINS` as each subdomain prefix (ie. `LETSENCRYPT_SUBDOMAINS=portainer,deluge,pihole`.

There are a number of sample proxy configuration files found in `~/.config/appdata/letsencrypt/nginx/proxy-confs/` and in most cases will just need the .sample removed from the filename. Currently not every applicable app has an example configuration and are still being tested.

Subfolder Example:
```
cp ~/.config/appdata/letsencrypt/nginx/proxy-confs/portainer.subfolder.conf.sample ~/.config/appdata/letsencrypt/nginx/proxy-confs/portainer.subfolder.conf
```
This will make Portainer available at `domain.com/portainer`

Subdomain Example:
```
cp ~/.config/appdata/letsencrypt/nginx/proxy-confs/portainer.subdomain.conf.sample ~/.config/appdata/letsencrypt/nginx/proxy-confs/portainer.subdomain.conf
```
and will enable the service at `portainer.domain.com`

Each time you change a proxy conf file you will need to restart the LetsEncrypt container:

`docker restart letsencrypt`

## Part 2
If you haven't forwarded ports for LE before container was setup, stop container, delete letsencrypt config folder, run `ds -c`, and you should be good to go.

If you are **not** using subdomains:
1. Blank out LETSENCRYPT_SUBDOMAINS in ~/.docker/compose/.env Like so:
```
LETSENCRYPT_SUBDOMAINS=
```
2. Fill in EMAIL, URL, like so:
```
LETSENCRYPT_EMAIL=user@domain.com
LETSENCRYPT_URL=appropriateaddress.com
```
3. `cp organizr.subfolder.conf.sample organizr.subfolder.conf` in ~/.config/appdata/letsencrypt/nginx/proxy-confs
4. Edit ~/.config/appdata/letsencrypt/nginx/site-confs/default and comment out the following (As shown):
```
#       location / {
#               try_files $uri $uri/ /index.html /index.php?$args =404;
#       }
```

## Why does my LetsEncrypt proxy configuration for (insert app here) say one port, but I have to access it with another port?
Generally speaking, your configuration _does_ point to the port you specify, which is correct. DockSTARTer sets your app to 8080 on your internal network, but docker has a docker network as well. On _that_ network your app runs on the default port for the app! That is how containers like LetsEncrypt and Organizr (For example) communicate.

## How do i automatically make http calls redirect to https for letsencrypt?

This change will make it so that if you type http://blahblah it will redirect to https://blahblah

1. Edit ~/.config/appdata/nginx/site-confs/default

2. Uncomment the relevant part of the file (see below)
```
# listening on port 80 disabled by default, remove the "#" signs to enable
# redirect all traffic to https
server {
	listen 80;
	listen [::]:80;
	server_name _;
	return 301 https://$host$request_uri;
}
```

3. Restart the letsencrypt container
`docker restart letsencrypt`
