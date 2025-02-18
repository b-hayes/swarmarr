# SwarmArr

A stack of docker containers from https://www.linuxserver.io/ to automate your media server.

This began because I really love Unraid, but I also needed to use the system as a desktop PC and
the Unraid GUI mode just does not work at all for me.

## Requirements

Any computer/platform that can run docker containers.

Install [Docker Engine](https://docs.docker.com/engine/install/) for Linux,
otherwise use [Docker desktop](https://docs.docker.com/get-started/get-docker/).


## Getting started

How to automount drives to specific paths op boot in Ubuntu.
https://askubuntu.com/questions/164926/how-to-make-partitions-mount-at-startup

I mount my big HD to `/array` (and plan to share this path with other drives in future).

Create some folders for all your stuff, mine looks like this:

 ```
/array/data/            # DATA_ROOT all services will have access here.
    - media/
        - movies/
        - tv/
    - downloads/
        - completed/
        - incomplete/
```
It's a lot easier if all services can access the same files and map to the same `/data/` path because they will 
communicate file paths with each other, eg: downloader telling radarr it saved a file for it to move into plex library.

 I put configs on the faster SSD, I use my home folder like so:
 
 ```
/home/user/configs      # CONFIG_ROOT each service will create a sub folder.
    # plex, radarr etc folders will be created automaticaly.
 ```

Edit the `.env` file to set the ROOT directories and timezone.

Start the swarm:
```SHELL
docker compose up -d
```
You should see a bunch of downloading and installing and then confirmation that all services are running.
You should see a bunch of folders appear in the config root for each service.
You want to back these up if you reinstall your OS.

Open the following links to configure each service:

 - Plex         http://localhost:32400/web
 - SABnzbd      http://localhost:8080
 - Radarr       http://localhost:7878
 - Sonarr       http://localhost:8989
 - Prowlarr     http://localhost:9696
 - Overseerr    http://localhost:5055
 - flaresolverr http://localhost:8191

 When browsing for paths they should all have `/data` path containing your DATA_ROOT files.

### Plex
- Pretty self-explanatory, access url to set up, as long as your paths are set correctly your good.

### SABnzbd
- Add your usenet account details click test and save.
- Edit the download paths and point them to your `/data/...` completed/incomplete paths.
- Edit usenet sever connections, default of  8 is a bit low.
- Open general settings and copy the API key, you'll need key for radarr and sonarr.

### Prowlarr
From the initial setup wizard:
- Set authentication method to Forms (login page).
- Add a username and password hit save.
- Go to Indexers (top one, not the one under settings) and add indexer.
  - Search for nzb geek or whatever you are using / have an account for or something free.
    - If selecting a free server check for notes about supporting flaresolverr connections.
  - Add your api key for your account
  - Add `movies` and `tv` tags.
  - Hit test and save if green tick appears.

Go setup [Radarr](#radarr) and [Sonarr](#sonarr) first and them come back here to finish.

#### Adding apps to prowlarr:
- Goto Settings->Apps, click add, click radarr or sonarr depending on what your here for.
- Add `movies` tag for radarr or `tv` tag for sonarr (optional?).
- Set servers to your computers local ip address.
Replace localhost and keep the rest. eg: `http://192.168.1.21:9696` instead of `http://localhost:9696`
- Set API key to whatever you copied from radar/sonarr.
- Click test and save if you see a green tick.

#### Adding Flaresolverr proxy.
- Go to Setting->Indexers click add
- Set name to Flaresolverr
- Add `movies` and `tv` tags.
- Set host to your IP address.
- Click test and save if you see a green tick.

### Flaresolverr
This stops Cloudflare blocking us from getting to our download.
Doesn't need configuring just [add it as proxy indexer in Prowlarr](#adding-flaresolverr-proxy).

### Radarr
- Choose forms (login page) as authentication method.
- Add a username and password and hit save.
- Add SABnzbd as your downloader:
    - Goto Settings->Download clients, click add, click SABnzbd.
    - Set Host to your computers local ip address.
    - Set API key the one you copied from SABnzbd general settings.
    - Add a `movies` tag (optional?).
- Go to Settings->General and copy the API key.
- Go to Settings->Media Management, scroll to the bottom and add a root folder.
  - add your `/data/.../movies` folder.
- Now go back to [Prowlarr app](#adding-apps-to-prowlarr) instruction to connect radarr.

### Sonarr
Basically just follow the [Radarr](#radarr) instructions but use `tv` tag and `/data/.../tv/` folder as media root.

## Sharing Folders on the network via SMB (on Ubuntu).

Sharing folders on linux for some reason is not a built-in thing for most distros.

Here is the entire list of steps I had to track down to get it working...

Install SMB
```shell
sudo apt-get install samba
```

Install file sharing options for Nautilus (the file explorer).
```shell
sudo apt install nautilus-share
```

Give yourself permissions to create shared folders or Nautilus will get errors.
```shell
sudo gpasswd -a $USER sambashare
```

Give yourself permission to share files you don't own (needed if you brought drives over from unraid).

Open the smb config:
```shell
sudo nano /etc/samba/smb.conf
```
Scroll to where it says `[ global ]` and write `usershare owner only = false` underneath like so:
```conf
[global]
usershare owner only = false
```

Log off and log back in.
Now you should be able to just right-click on a folder and see sharing options.

## Creating a user for share folders only
Ubuntu does a windows like setup where, a user account must exist on the machine to get access to stuff.
But you can create another smb share manually, but it's a pain in the ass.
Looking for a better option than using command line for this (and might build one).

## Enabling access via computer name and mDNS routing.
Accessing your server via computer-name.local as unraid did is common, but it's blocked by default on Ubuntu.
I actually don't know how, but setting up smb file sharing above will enable it you.

## Spanning multiple Drives with redundancy

The key feature I liked about Unraid was its storage solution.
You can replicate it with the following tools.

[SnapRaid](https://www.snapraid.it/) for parity backup
and [mergerfs](https://github.com/trapexit/mergerfs/blob/master/README.md)
for sharing mounts across drives.

You do not need this, but its worth looking into after you get setup.
Good article on this here: https://perfectmediaserver.com/02-tech-stack/snapraid/

## Stopping the swarm

```SHELL
docker compose down
docker compose down -v # -v to delete the containers volume as well.
```

# Backup and restore.
Your configs being on the faster ssd in your home folder could get wiped out from an OS reinstall, so I recommend
having a regular backup of the configs folder copied to the array jic.

```shell
sudo crontab -e # opens scheduled tasks config on linux
```
I typically add the following:
```config
# backup configs every day at 3pm
0 15 * * * cp -rp /home/brad/repo/configs /array/backup/configs-$(date +'%Y-%m-%d-%H-%M-%S')
```

To restore your setup on a new OS/PC just install docker again, copy the configs back where it was before,
and run docker compose up and your services should resume as they were.