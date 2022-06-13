# 🎞️ Ansible-homemedia 
[![MIT license](https://img.shields.io/badge/License-MIT-blue.svg)](https://lbesson.mit-license.org/)

Ansible playbook to install a private media server. Goal was to provide a reasonably light, maintainable, automated and easy to setup solution with sensible defaults. It assumes every application is served through dedicated subdomains so have your DNS properly setup.

### Features

* HTTPS Reverse proxy, authentification, authorization and apps portal through [Caddy](https://caddyserver.com/) with preconfigured secured routes. 
* Automatic updates of non major versions for SemVer containers with [What's up Docker](https://fmartinou.github.io/whats-up-docker/#/). By default, it cleans up old images and updates everyday at 4 A.M.
* Automatic firewall configuration and secure SSH daemon setup
* Monitoring of Docker using [Portainer](https://www.portainer.io)
* Full use of the *Arr stack and [Deluge](https://deluge-torrent.org) to manage the downloads and subtitles.
* Proper setup of the filesystem and permissions to enable hardlinks and atomic moves
* Streaming and organization of your medias using [Plex](https://www.plex.tv)

### Optionals

These are not enabled by default, but you can change that by editing the ansible `group_vars/all` file prior to starting the playbook.

* Multi-factor authentication (MFA) upon auth portal login using [Caddy security plugin](https://authp.github.io/docs/authenticate/mfa). On first auth portal launch it'll ask you to setup MFA.
* [Flaresolverr](https://github.com/FlareSolverr/FlareSolverr) to avoid Cloudflare's challenges on specific trackers. Prowlarr can natively integrate with it.
* [Overseerr](https://overseerr.dev/) to manage media requests. Despite being behind Caddy for HTTPS, it does not require Caddy authentication. Instead it is linked to Plex server instance which will manage external users for you.
* Allows for encrypted and remote backups of persistent container data with [Duplicati](https://www.duplicati.com/) and/or [Duplicacy](https://duplicacy.com/). Both offer nice web UI but I suggest going for a personal Duplicacy license if you can because, although free, Duplicati error reports aren't the best.

## ⛩️ Architecture

![arch](https://user-images.githubusercontent.com/1922364/170122807-79b6aa7c-869f-4475-835c-ff9fcee62b28.png)

<sub>*Folder icons are a more or less simplifed view of volumes mapping*</sub>

## ⚙️ Usage 

1) Replace the placeholders in the hosts inventory file to use your server and SSH user
2) Replace the placeholders in the `group_vars/all` to match your setup
3) Run the playbook `ansible-playbook homemedia-setup.yml -K`
4) SSH to your server and run `docker logs caddy` to grab default login and password ([see bottom page](https://authp.github.io/docs/authenticate/local/local)) for the Caddy portal. Go to your domain to login, change password and go to `portainer.{domain.com}` to make sure everything is up and running.
5) Start a SSH tunnel `ssh -g -L 32400:localhost:32400 -N {domain.com}`, claim your PLEX server at `localhost:32400/web`, map the `/homemedia/medias` folders as you want and enable remote access with port 32400. You should then be able to reach your PLEX server from any external PLEX client.
6) Configure Deluge and the *Arr stack with the help of https://wiki.servarr.com and https://trash-guides.info/
7) (Optional) Configure Duplicati/Duplicacy automated backups (see FAQ)
8) (Optional) Configure Overseerr

## 🤔 FAQ

### Why is there no VPN ?

Because I don't need it, but one could probably add it easily. Just add and configure a new OpenVPN container in `docker-compose.yml`, route the approriate traffic through it and that's probably it ? You could also add an Ansible VPN role to help you with the setup of the container as well. PR welcome if you can make it optional and non default.

### How do backups work exactly ?

Most of the docker images used in the project are saving their important files in one or several specific folders. By default this is all mapped on the host in the directory tree `app_dir` for every application. You should configure Duplicati/Duplicacy to backup this directory to another remote location, as well as others if you so wish. Doing so if your machine crashes and burn, you can just run the playbook again and restore your remote backup.

One thing to keep in mind is that, unless you decide to run  as `root` the container doing the backup, it won't restore `root` owned files and directories so make sure you exclude them from your backup strategy. That includes `{{ app_dir }}/caddy` and  `{{ app_dir }}/wud` paths, as their containers has to run as `root`.

### Can other people access my applications ?

If we're talking PLEX server yes, but otherwise no. By default in this Caddy security auth portal there's only one user, and that's your admin user. I wouldn't want anyone else accessing the apps. If you need a request system however you can just enable Overseerr.

### I have a problem with setting up XYZ application !

Kindly RTFM.

### Why all the latest tags ?

I know this is Ansible and all but the point wasn't to be idempotent. It was to just run the playbook and have it install the latest versions of everything without having to maintain said versions. Of course the downside is you could have unwanted major version updates by running it several times. Not much of a problem to me since you probably won't do that after the initial setup. If you really need to you can just disable the two tasks calling `docker compose` or outright disabling the `compose` role.

### I already have an existing enviromnent, can I use this ?

Possible but not advised because this playbook was made for running on a fresh Ubuntu install. You need to watch Ansible and Docker versions as well as having a user who can write in `/` (or change `root_dir`). The playbook do use newish syntax, so Ansible >= 2.5 for the controller should suffice. For docker and compose, it is automatically installed by the playbook. However if you already have it then disable the `docker` role. Know the playbook assumes at least Compose V2, so if you're running older versions you should add back the hyphens in the `compose` role tasks. In addition be wary of the firewall and filesystem role, you may have to tweak a few things. YMMV because I haven't tested any of this, but you could using the `Vagrantfile`.

### Can I get a wildcard certificate ?

Not out of the box because that would force me to take into account everyone's DNS provider. Good news is that you can do it yourself [fairly easily](https://caddyserver.com/docs/caddyfile/patterns#wildcard-certificates) modifying Caddy's Dockerfile and Caddyfile.

## 📝 Testing

There's a `Vagrantfile` available if you'd like to contribute something or just mess around in a headless system. Just make sure you have [Vagrant](https://www.vagrantup.com/) installed as well as a provider (VirtualBox recommended). Then clone the repository and run `vagrant up` in the root directory. This will run the playbook without any of the opt-ins against a local VM in which you can `vagrant ssh` to snoop around after if you want.
