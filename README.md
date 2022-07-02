<!--
Maintainer:   jeffskinnerbox@yahoo.com / www.jeffskinnerbox.me
Version:      0.0.1
-->


<div align="center">
<img src="http://www.foxbyrd.com/wp-content/uploads/2018/02/file-4.jpg" title="These materials require additional work and are not ready for general use." align="center">
</div>


----


```bash
sudo MQ_PORT=1883 MQ_HOME='/home/jeff/src/docker-containers/home-assistant' docker-compose --file ./mosquitto-docker-compose.yml up -d
sudo MDB_PORT=3306 AD_PORT=8080 MYSQL_ROOT_PASSWORD='password' MYSQL_PASSWORD='password' MDB_HOME='/home/jeff/src/docker-containers/home-assistant' docker-compose --file ./mariadb-docker-compose.yml up -d
sudo HA_PORT=8123 HA_HOME='/home/jeff/src/docker-containers/home-assistant' docker-compose --file ./homeassistant-docker-compose.yml up -d
sudo NR_PORT=1880 NR_HOME='/home/jeff/src/docker-containers/home-assistant' docker-compose --file ./nodered-docker-compose.yml up -d
```


----


# Home Assistant
To install a Docker version of Home Assistant,
you can just follow the instructions on the [Home Assistant website][01].
Other useful `docker-compose` file examples are [here][02].

```yaml
version: '3'
services:
  homeassistant:
    container_name: homeassistant
    image: "ghcr.io/home-assistant/home-assistant:stable"
    restart: unless-stopped
    privileged: true
    network_mode: host   # shares host networking with container, enables device discovery (zeroconf/mDNS and UPnP devices)
    #devices:
      #- /dev/ttyUSB0:/dev/ttyUSB0
      #- /dev/ttyUSB1:/dev/ttyUSB1
      #- /dev/ttyACM0:/dev/ttyACM0
    ports:
      - ${HA_PORT}:8123                    # port to access webui
    environment:
      - PUID=1000                          # user id
      - PGID=1000                          # group id
      - TZ="America/NewYork"               # set time zone
    volumes:
      - ${HA_HOME}/config:/config          # home assistant config storage path
      - ${HA_HOME}/shared:/shared
      - /etc/localtime:/etc/localtime:ro   # sets containers time
```

You'll often need to include other capabilities for Home Assistant,
such as Node-RED, MariaDB, Mosquitto, HASS Configurator.
Here is a Docker Compose file that provides those utilities:

```yaml
version: '3'
services:
  homeassistant:
    container_name: home-assistant
    image: "ghcr.io/home-assistant/home-assistant:stable"
    restart: unless-stopped
    privileged: true
    network_mode: host   # shares host networking with container, enables device discovery (zeroconf/mDNS and UPnP devices)
    #devices:
      #- /dev/ttyUSB0:/dev/ttyUSB0
      #- /dev/ttyUSB1:/dev/ttyUSB1
      #- /dev/ttyACM0:/dev/ttyACM0
    ports:
      - ${HA_PORT}:8123                    # port to access webui
    environment:
      - PUID=1000                          # user id
      - PGID=1000                          # group id
      - TZ="America/NewYork"               # set time zone
    volumes:
      - ${HA_HOME}/config:/config          # home assistant config storage path
      - ${HA_HOME}/shared:/shared
      - /etc/localtime:/etc/localtime:ro   # sets containers time

https://iotechonline.com/home-assistant-install-with-docker-compose/
```

```bash
# start the docker container
sudo HA_PORT=8123 HA_HOME='/home/jeff/src/docker-containers/home-assistant' docker-compose --file ./homeassistant-docker-compose.yml up -d

# check for any errors during the install via
sudo docker logs home-assistant
```

Once the Home Assistant Container is running,
Home Assistant can be accessible using `http://localhost:8123`.

## Removing Images & Containers
If you run into problems, kill Home Assistant
and clean-up any unused resources
(i.e. images, containers, volumes, and networks)
that are dangling (not tagged or associated with a container).
This is shown below:

```bash
# kill the container
sudo docker kill home-assistant

# remove the container
sudo docker rm home-assistant

# remove the container image
sudo docker rmi ghcr.io/home-assistant/home-assistant

# list any dangling images
sudo docker images -f dangling=true

# removes all unused containers, networks, images (both dangling and unreferenced), and optionally, volumes
sudo docker system prune
```

Source:

* [Home Assistant: Docker Compose][01]
* [DockerHub: linuxserver/homeassistant][02]
* [Home Assistant install with docker-compose][03]

## Exposing Devices
In order to use Z-Wave, Zigbee or other integrations that require access to devices,
you need to map the appropriate device into the container.
Ensure the user that is running the container has the correct privileges to access the `/dev/tty*` file,
then add the device mapping to your container instructions:

```bash
docker run ... --device /dev/ttyUSB0:/dev/ttyUSB0 ...
```
# Updating Container
Since data is preserved outside of the containera via the volumes,
updating the base container image is now as simple as:

```bash
sudo docker pull nodered/node-red
sudo docker stop mynodered
sudo docker rm mynodered
sudo docker run -it -p 1880:1880 -v node_red_data:/data --name mynodered nodered/node-red
```


-----



# Home Assistant Configurator (aka hass-configurator)
The HASS-Configurator is a small webapp (i.e. you access it via web browser)
that provides a filesystem browser and text editor to allow easy configuration of Home Assistant.
It uses the [Ace editor][05], which supports syntax highlighting for various code/markup languages.
YAML files (the default language for Home Assistant configuration files)
will be automatically checked for syntax errors while editing.

>**NOTE:** Ace can be made to matches the features and performance of
>native editors such as Sublime, Vim and TextMate.
https://stackoverflow.com/questions/15485153/enable-vim-mode-in-gist-ace-editor

```yaml
```


-----



# Node-RED

```yaml
version: '3'
services:
  nodered:
    container_name: node-red
    image: nodered/node-red:latest
    restart: unless-stopped
    ports:
      - ${NR_PORT}:1880
    environment:
      TZ: "America/NewYork"       # set time zone
    volumes:
      - ${NR_HOME}/node-red:/data
    #depends_on:
      #- homeassistant
      #- mosquitto
```

Node-RED will persist its directory inside the container via the host directory system.
To allow access to this host directory,
the `node-red` user (which defaults to uid is 1000) inside the container,
[must have the same uid as the owner of the host directory][04].
To assure this is the case, prior to running `docker-compose`,
you need to create the directory you plan to persist the Node-RED container:

```bash
# create the directory for persisting node-red data
cd ~/src/docker-containers/home-assistant
mkdir node-red

# now instantiate the node-red container
sudo NR_PORT=1880 NR_HOME='/home/jeff/src/docker-containers/home-assistant' docker-compose --file ./nodered-docker-compose.yml up -d
```

Sources:

* [Node-Red in Docker with Home Assistant!!](https://www.youtube.com/watch?v=fxo5-iiwZXk)
* [Node Red for Beginners - 2021 Edition (Using Home Assistant)](https://www.youtube.com/watch?v=hBEb_FCLRU8)



-----



# Mosquitto

```yaml
version: '3'
services:
  mosquitto:
    image: eclipse-mosquitto:latest
    container_name: mosquitto
    #user: ${PUID}:${PGID}
    user: "1000:1000"
    restart: unless-stopped
    ports:
      - ${MQ_PORT}:1883
      - 1884:1884
    volumes:
      - ${MQ_HOME}/mosquitto/config:/mosquitto/config
      - ${MQ_HOME}/mosquitto/data:/mosquitto/data
      - ${MQ_HOME}/mosquitto/log:/mosquitto/log
    environment:
      TZ: America/NewYork       # set time zone
```

Mosquitto will persist its directory inside the container via the host directory system.
You should set up persistent data directories and the Mosquitto basic configuration file
(`mosquitto.conf`) before starting your container.

```bash
# create the directory for persisting mosquitto data
cd ~/src/docker-containers/home-assistant
mkdir -p mosquitto/config
mkdir -p mosquitto/data
mkdir -p mosquitto/log

# create basic mosquitto configuration file
cat <<EOF > mosquitto/config/mosquitto.conf
persistence true
persistence_location /mosquitto/data/

user mosquitto

# Listen on all interfaces
listener 1883

#Allow connection without authentication
allow_anonymous true

log_dest file /mosquitto/log/mosquitto.log
log_dest stdout
EOF

# now instantiate the mosquitto container
sudo MQ_PORT=1883 MQ_HOME='/home/jeff/src/docker-containers/home-assistant' docker-compose --file ./mosquitto-docker-compose.yml up -d
```

To test the installation, install the Mosquitto client,
subscribe to a topic in another terminal session,
and send a message to yourself.
To do this, you will need the IP address of the Mosquitto borker you just installed.
You can get this from Portainer.

```bash
# install the mosquitto client package
sudo apt install mosquitto-clients

# subscribe to a topic on the mosquitto broker
mosquitto_sub -h 172.24.0.4 -t "test"

# in another terminal window, send a test message
mosquitto_pub -h 172.24.0.4 -t "test" -m "Hello World"
```

As it is currently set up, any client can connect to your broker to publish
and subscribe to any topic.
To add additional security, checkout this [webpost][06].

Sources:

* [MQTT With Home Assistant Using Docker & Mosquitto][06]
* [DockereHub: eclipse-mosquitto](https://hub.docker.com/_/eclipse-mosquitto)



-----



# MariaDB
MariaDB is a backward compatible, drop-in replacement of the MySQL database server.

Adminer (formerly phpMinAdmin) is a full-featured database management tool written in PHP. Conversely to phpMyAdmin, it consist of a single file ready to deploy to the target server. Adminer is available for MySQL, MariaDB, PostgreSQL, SQLite, MS SQL, Oracle, Elasticsearch, MongoDB and others via plugin.




[01]:https://www.home-assistant.io/installation/raspberrypi#docker-compose
[02]:https://hub.docker.com/r/linuxserver/homeassistant
[03]:https://iotechonline.com/home-assistant-install-with-docker-compose/
[04]:https://nodered.org/docs/getting-started/docker#using-a-host-directory-for-persistence-bind-mount
[05]:https://ace.c9.io/
[06]:https://hometechhacker.com/mqtt-home-assistant-using-docker-eclipse-mosquitto/
[07]:
[08]:
[09]:
[10]:

