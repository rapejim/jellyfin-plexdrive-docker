# ***Jellyfin Server and Plexdrive 🐳***

<div align="center"><img src="https://raw.githubusercontent.com/rapejim/jellyfin-plexdrive-docker/master/images/banner.png" width="50%"></div>

Combine the power of **Jellyfin Server** *(hereinafter Jellyfin)* with the media files of your Google Drive account *(or a Team Drive)* mounted it by **Plexdrive**.

Based on [Linuxserver Jellyfin image for Docker](https://fleet.linuxserver.io/image?name=linuxserver/jellyfin) and installed inside [Plexdrive v.5.1.0](https://github.com/plexdrive/plexdrive)<br>
*Inspired on my other repository https://github.com/rapejim/pms-plexdrive-docker.* <br>
***IMPORTANT:*** *All options are inherited from the Linuxserver Jellyfin container. [Refer to Linuxserver documentation for more info](https://docs.linuxserver.io/images/docker-jellyfin).* 
<br>
<br>
<br>

## ***Prerequisites***
---

You must have your own `Client ID` & `Client Secret` to configure plexdrive. If you don't have it, you can follow any internet guide, for example:
- [English](https://github.com/Cloudbox/Cloudbox/wiki/Google-Drive-API-Client-ID-and-Client-Secret)
- [Spanish](https://www.uint16.es/2019/11/04/como-obtener-tu-propio-client-id-de-google-drive-para-rclone/)

Or you can use the configuration files from a previous plexdrive installation (the `config.json` and `token.json` files, preferably not reusing the `cache.bolt`, it is better that this installation generates a new one).
<br>
<br>
<br>

## ***Example run commands***
---

### **Minimal** example run command *(host network)*:

```
docker run --name Jellyfin -d \
    --net=host \
    -e PUID=${UID} \
    -e PGID=$(id -g) \
    -e TZ="Europe/Madrid" \
    -v /docker/jellyfin-plexdrive/config:/config \
    -v /docker/jellyfin-plexdrive/transcode:/transcode \
    --privileged \
    --cap-add MKNOD \
    --cap-add SYS_ADMIN \
    --device /dev/fuse \
    --restart=unless-stopped \
    rapejim/jellyfin-plexdrive-docker
```
***NOTE:*** *You must replace `Europe/Madrid` for your time zone and `/docker/jellyfin-plexdrive/...` for your own path (if not use this folder structure). If you have config files (`config.json` and `token.json`) from previous installation of plexdrive, place it on `/docker/jellyfin-plexdrive/config/.plexdrive` folder.* 
<br>
<br>
<br>

### **Advanced** example run command *(bridge network)*:

```
docker run --name Jellyfin -h Jellyfin -d \
    -p 8096:8096 \
    -e PUID=${UID} \
    -e PGID=$(id -g) \
    -e TZ="Europe/Madrid" \
    -v /docker/jellyfin-plexdrive/config:/config \
    --privileged \
    --cap-add MKNOD \
    --cap-add SYS_ADMIN \
    --device /dev/fuse \
    --restart=unless-stopped \
    rapejim/jellyfin-plexdrive-docker
```
***NOTE:*** *You must replace `Europe/Madrid` for your time zone and `/docker/jellyfin-plexdrive/...` for your own path (if not use this folder structure). If you have config files (`config.json` and `token.json`) from previous installation of plexdrive, place it on `/docker/jellyfin-plexdrive/config/.plexdrive` folder.*
<br>
<br>
<br>

## ***First usage and initial config***
---
On the first run of container (without config files of previous installation) you must enter inside container console, copy-paste and run this command:
```
plexdrive mount -c ${HOME}/${PLEXDRIVE_CONFIG_DIR} --cache-file=${HOME}/${PLEXDRIVE_CONFIG_DIR}/cache.bolt -o allow_other ${PLEXDRIVE_MOUNT_POINT} {EXTRA_PARAMS}
```
This command start a config wizzard:
- It asks you for your `Client ID` and `Client Secret`
- Shows you a link to log in using your Google Drive account (which you used to get that `Client ID` and `Client Secret`).
- The web show you a token that you must copy it and paste on terminal.
- When you complete it, Plexdrive start caching your Google Drive account files on second plane, no need to wait for Plexdrive to complete its initial cache building process on this console, now you have the `config.json` and `token.json` created and can exit from terminal (*Cntrl + C* and `exit`).
<br>
<br>
<br>

## ***Parameters***
---

Those are not required unless you want to preserve your current folder structure or maintain special file permissions.

- `PLEXDRIVE_CONFIG_DIR` Sets the name of Plexdrive config folder found within PMS config folder. Default is `.plexdrive`.
- `PLEXDRIVE_MOUNT_POINT` Sets the name of internal Plexdrive mount point. Default is `/home/Plex`.
- `CHANGE_PLEXDRIVE_CONFIG_DIR_OWNERSHIP` Defines if the container should attempt to correct permissions of existing Plexdrive config files.
- `PUID` and `PGID` Sets user ID and group ID for `Jellyfin` user. Useful if you want them to match those of your own user on the host.
- `EXTRA_PARAMS` Add more advanced parameters for plexdrive to mount initial command. You can use, for example:
  - `--drive-id=ABC123qwerty987` for **Team Drive** with id `ABC123qwerty987`
  - `--root-node-id=DCBAqwerty987654321_ASDF123456789` for a mount only the sub directory with id `DCBAqwerty987654321_ASDF123456789`
  - *[... plexdrive documentation for more info ...](https://github.com/plexdrive/plexdrive#usage)*
  -  **IMPORTANT:** *Not allowed "`-v` `--verbosity`", "`-c` `--config`", "`--cache-file`" or "`-o` `--fuse-options`" parameters, because are already used.*
<br>
<br>

***REMEMBER:*** *All options from the Linuxserver Jellyfin container are inherited. [Refer to Linuxserver documentation for more info](https://docs.linuxserver.io/images/docker-jellyfin).*
<br>
<br>
<br>

## ***Tags***
---

Tags correspond to those of the official PMS Docker container:

- `nightly` — Nightly Jellyfin releases - nightly linuxserver baseimage.
- `latest` — Stable Jellyfin releases - latest linuxserver baseimage.
