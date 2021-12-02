# ***Jellyfin Server and Plexdrive 🐳***

<div align="center"><img src="https://raw.githubusercontent.com/rapejim/jellyfin-plexdrive-docker/develop/images/banner.png" width="50%"></div>

Combina el poder de **Jellyfin Server** *(en adelante Jellyfin)* con los archivos multimedia de tu cuenta de Google Drive (o en una [Unidad Compartida](https://support.google.com/a/users/answer/9310156?hl=es)) montados con [**Plexdrive**](https://github.com/plexdrive/plexdrive).

Basado en la [imagen de Jellyfin de Linuxserver](https://fleet.linuxserver.io/image?name=linuxserver/jellyfin) e instalado dentro [Plexdrive v.5.1.0](https://github.com/plexdrive/plexdrive).
*Inspirada en mi otro repositorio https://github.com/rapejim/pms-plexdrive-docker.* <br>

***IMPORTANTE:*** *Se heredan todas las opciones de la imagen Jellyfin de Linuxserver. [Documentación de Linuxserver para más información](https://docs.linuxserver.io/images/docker-jellyfin).*
<br/><br/>Puedes leer este documento en otros idiomas: [English](https://github.com/rapejim/pms-plexdrive-docker/blob/develop/README.md), [Español](https://github.com/rapejim/pms-plexdrive-docker/blob/develop/README.ES.md)

## *Prerrequisitos*

---

Debes tener tu propio identificador de cliente (`Client ID`) y secreto de cliente (`Client Secret`) para configurar Plexdrive. En caso no cuentes con ello, puedes seguir alguna guía que encuentres por Internet, por ejemplo:

- [Español](https://www.uint16.es/2019/11/04/como-obtener-tu-propio-client-id-de-google-drive-para-rclone/)
- [English](https://github.com/Cloudbox/Cloudbox/wiki/Google-Drive-API-Client-ID-and-Client-Secret)

O puedes usar los archivos `config.json` y ` token.json` de una instalación previa de Plexdrive, aunque en este caso es preferible no reutilizar el archivo `cache.bolt` para que se genere uno nuevo.
<br/><br/>

## *Ejemplos de comandos de ejecución*

---

##### Línea de comandos con red host

```bash
docker run --name jellyfin -d \
    --net=host \
    -e TZ="Europe/Madrid" \
    -e PUID=${UID} \
    -e PGID=$(id -g) \
    -v /docker/jellyfin/config:/config \
    # -v /opt/vc/lib:/opt/vc/lib \ # Opcional
    --privileged \
    --cap-add MKNOD \
    --cap-add SYS_ADMIN \
    --device /dev/fuse \
    --restart=unless-stopped \
    rapejim/jellyfin-plexdrive-docker
```

##### Línea de comandos con red bridge

```bash
docker run --name jellyfin -h Jellyfin -d \
    -p 8096:8096/tcp \
    # -p 8920:8920/tcp \ # Opcional HTTPS
    # -p 1900:1900/udp \ # Opcional DLNA
    # -p 7359:7359/udp \ # Opcional LAN discovery
    -e TZ="Europe/Madrid" \
    -e PUID=${UID} \
    -e PGID=$(id -g) \
    -v /docker/jellyfin/config:/config \
    # -v /opt/vc/lib:/opt/vc/lib \ # Opcional
    --privileged \
    --cap-add MKNOD \
    --cap-add SYS_ADMIN \
    --device /dev/fuse \
    --restart=unless-stopped \
    rapejim/jellyfin-plexdrive-docker
```



##### Docker-compose con red host

```yaml
version: '3.5'
services:
  jellyfin:
    container_name: jellyfin
    image: rapejim/jellyfin-plexdrive-docker # https://hub.docker.com/r/rapejim/jellyfin-plexdrive-docker
    restart: unless-stopped
    privileged: true
    network_mode: host
    volumes:
      - /docker/jellyfin/config:/config
      # - /opt/vc/lib:/opt/vc/lib # Opcional
    environment:
      TZ: Europe/Madrid
      PUID: '1000'
      PGID: '1000'
    cap_add:
      - MKNOD
      - SYS_ADMIN
    devices:
      - "/dev/fuse"
```

##### Docker-compose con red bridge

```yaml
version: '3.5'
services:
  jellyfin:
    container_name: jellyfin
    hostname: Jellyfin
    image: rapejim/jellyfin-plexdrive-docker # https://hub.docker.com/r/rapejim/jellyfin-plexdrive-docker
    restart: unless-stopped
    privileged: true
    network_mode: bridge
    ports:
      - 8096:8096
    #   - 8920:8920 # Opcional HTTPS
    #   - 7359:7359/udp # Opcional LAN discovery
    #   - 1900:1900/udp # Opcional DLNA
    volumes:
      - /docker/jellyfin/config:/config
      # - /opt/vc/lib:/opt/vc/lib # Opcional
    environment:
      TZ: Europe/Madrid
      PUID: '1000'
      PGID: '1000'
    cap_add:
      - MKNOD
      - SYS_ADMIN
    devices:
      - "/dev/fuse"
```



***NOTA:*** *Debes reemplazar `Europe/Madrid` por tu zona horaria y las rutas de los volúmenes `/docker/jellyfin/...` por tus propias rutas (si no usas la misma estructura de carpetas). Si tienes archivos de configuración (`config.json` y ` token.json`) de una instalación anterior de Plexdrive, colócalos previamente en la carpeta `docker/jellyfin/config/.plexdrive`.*
<br>
<br>
<br>

## *Primer uso y configuración inicial*

---

En la primera ejecución del contenedor (si no tienes archivos de configuración de una instalación previa) debes entrar en la consola del contenedor, copiar, pegar y ejecutar el siguiente comando:

```bash
plexdrive mount -c ${HOME}/${PLEXDRIVE_CONFIG_DIR} --cache-file=${HOME}/${PLEXDRIVE_CONFIG_DIR}/cache.bolt -o allow_other ${PLEXDRIVE_MOUNT_POINT} {EXTRA_PARAMS}
```

Este comando iniciará un asistente de configuración:

- Primero solicitará tus `Client ID` y `Client Secret`
- Te mostrará un enlace para iniciar sesión con tu cuenta de Google Drive (la misma de los `Client ID` y `Client Secret`).
- El sitio web del enlace anterior, te mostrará un token que debes copiar y pegar en el terminal.
- Cuando completes el proceso, Plexdrive comienzará a almacenar en caché los archivos de tu cuenta de Google Drive en el segundo plano. No es necesario esperar a que Plexdrive complete su proceso inicial de creación de caché en esta consola. Ahora que los archivos `config.json` y ` token.json` fueron creados, puedes salir de la terminal (*Ctrl+C*).

<br>
<br>
<br>

## *Parámetros*

---

Estos parámetros no son necesarios, solo si deseas usar una estructura de carpetas actual o permisos de archivo especiales.

- `PLEXDRIVE_CONFIG_DIR` Establece la carpeta de configuración de Plexdrive dentro de la carpeta de configuración de PMS. Valor predeterminado `.plexdrive`.

- `PLEXDRIVE_MOUNT_POINT` Establece el nombre del punto de montaje interno de Plexdrive.
  Valor predeterminado  `/home/Plex`.

- `CHANGE_PLEXDRIVE_CONFIG_DIR_OWNERSHIP` Determina si el contenedor debe intentar corregir los permisos de los archivos de configuración de Plexdrive existentes.

- `PUID` y `PGID` Establece el ID de usuario y el ID de grupo para el usuario de `Plex`. Útil si deseas que coincidan con los de su propio usuario en el ordenador.

- `EXTRA_PARAMS` Agrega parámetros avanzados a Plexdrive  para usar en el comando inicial de montaje. Por ejemplo:

  - `--drive-id=ABC123qwerty987` para montar la **Unidad Compartida** con el identificador `ABC123qwerty987`

  - `--root-node-id=DCBAqwerty987654321_ASDF123456789` para un montar solo el subdirectorio con el identificador `DCBAqwerty987654321_ASDF123456789`

  - *[... Documentación de Plexdrive para más información ...](https://github.com/plexdrive/plexdrive#usage)*

  - **IMPORTANTE:** *No está permitido utilizar los parámetros "`-v` `--verbosity`", "`-c` `--config`", "`--cache-file`", ni tampoco "`-o` `--fuse-options`", porque ya se usan internamente.*

    <br/>
    <br/>

***RECUERDA:*** *Se heredan todas las opciones de la imagen Jellyfin de Linuxserver. [Documentación de Linuxserver para más información](https://docs.linuxserver.io/images/docker-jellyfin).*
<br>
<br>
<br>

## ***Etiquetas de Docker***
---

Las etiquetas corresponden a las de la imagen Jellyfin de Linuxserver:

- `nightly` — versiones nightly de Jellyfin - imagen base nightly de linuxserver.
- `latest` — versiones estables de Jellyfin - imagen base latest de linuxserver.
