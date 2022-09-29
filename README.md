# My Docker notes

* `docker create centos`

## Docker `attach` and `exec`

### `docker attach [OPTIONS] CONTAINER`

* `docker attach myContainer` # connect to stdout/stdin/stderror of the main proccess

### `docker exec [OPTIONS] CONTAINER COMMAND`

* `docker exec -it myContainer /bin/bash`

* `docker exec -it myContainer ls /home/`

* `docker exec -it myContainer ping 8.8.8.8`

* `docker exec -it -w /home myContainer ls`

* `docker exec -it -e myvar=val1 myContainer bash`

---

## Default ENV variables

`HOME`, `HOSTNAME`, `PATH` and `TERM`

---

## General commands

### pull

### create

### run

* `docker run -it -e myvar=val1 --name mycent centos:latest`
* `docker run --rm ...` remove the container if the main proccess finished

### tag

* `docker tag image_id name`

### inspect

* `docker inspect Container`

### cp

* `docker cp file_src Container:/PATH`

* `docker cp Container:/PATH dest`

### stop

* `docker stop [OPTIONS] CONTAINER [CONTAINER...]`

* `docker stop -t 20 myContainer`

### start

* `docker start CONTAINER`

* `docker start -a CONTAINER` # to having stdout/stderror

* `docker start -i CONTAINER` # to having stding


### restart

* `docker restart CONTAINER`

* `docker restart -t 20 myContainer`

### Wait

* `docker wait cent1` # wait until main proccess finished

### kill

* `docker kill [CONTAINER...]`

### pause/unpause

* `docker pause cent1`
* `docker unpause cent1`

### rename 

* `docker rename mycent my_newcent`

### commit

* `docker commit container image:version` # create image from container

**not recommended to create image via this approach. best practice is create image via docker file**

* `docker commit --change='CMD ["/bin/sh"]' ...` # change the main process


### save/load

* `docker save -o mycentos.tar.gz mycentos:v1` # create an archive from image

* `docker load -i archive.tar.gz`

* `docker load < archive.tar.gz`

### export/import

* `docker export mycent > mycent.tar` # create an image from container

* `docker import mycent.tar`

**dangle image** will be created --> `docker tag image_id tag:v`

---

## Docker Logs/Events

Events are related to docker host while the logs are related to containers.

* `docker logs -ft mycent`

* `docker events`
* `docker events --since ...`
* `docker events --until ...`
* `docker events --filter ...`

---

## Docker Logging

* `docker info --format '{{.LoggingDriver}}'`    
by default `json-file`

* you can change along the system `/etc/docker/daemon.json`
```json
{"log-driver": "syslog"}
```
* `log-opts`, `max-size`, `max-file`, `labels`,....

* `docker run -it --log-driver logging_driver centos` # change the log driver per container

* `ducker inspect -f '{{.HostConfig.LogConfig.Type}}' container`

### Approches
* blocking: direct delivery from container to driver (default)
* non-blocking: store --> buffer --> send to driver

`docker run -ot --log-opt mode=non-blocking --log-opt max-buffer-size=4m cents`

### log location

`/var/lib/docker/containers/e8e19.../e8e19...log`

---

## Docker Volume
* To persist the data.
* Backup
* Performance

### 1. Volume

An approach to store some data of the container outside of the container. (In the disk, storage, HDD, ...)

* Default path of the volume on Linux: `/var/lib/docker/volumes/`
* non-docker processes should not modify this path.
* Best way to persist data in docker.

#### 4. Commands

* `docker volume create myvol`
* `ls /var/lib/docker/volumes/ -la`
```
total 36
drwx-----x  3 root root  4096 Sep 15 17:51 .
drwx--x--- 13 root root  4096 Sep 15 16:39 ..
brw-------  1 root root  8, 2 Sep 15 16:39 backingFsBlockDev
-rw-------  1 root root 32768 Sep 15 17:51 metadata.db
drwx-----x  3 root root  4096 Sep 15 17:51 myvol
```

* `docker volume create --name myvol2 --label testvol`
* `$ sudo docker volume ls`
```
DRIVER    VOLUME NAME
local     myvol
local     myvol2
```

* `docker run -d -v myvol:/data --name cent1 centos`
`touch /home/test`

    in docker host: `ls /var/lib/docker/volumes/myvol`

* `docker run -d -v /home:/data centos`
sharing the `/home` of the docker host with container.

* `docker volume ls -f name=m` # regex base
```
DRIVER    VOLUME NAME
local     myvol
local     myvol2
local     myvol3
```

* `docker inspect myvol`
```
[
    {
        "CreatedAt": "2022-09-15T17:59:56+04:30",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/myvol/_data",
        "Name": "myvol",
        "Options": {},
        "Scope": "local"
    }
]
```

* `docker run -it --name cent6 -v myvol:/data -v myvol2:/data2:ro centos`

* `docker ps -a --filter volume=myvol2`
```
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS                      PORTS     NAMES
5a25a8813c73   centos    "/bin/bash"   About a minute ago   Exited (0) 58 seconds ago             cent61
96cb39ebf95a   centos    "/bin/bash"   2 minutes ago        Exited (0) 2 minutes ago              cent6
```

* `docker inspect --format="{{.Mounts}}" 354b9d691678`
```
[{volume myvol /var/lib/docker/volumes/myvol/_data /home local z true }]
```

* `docker volume create --driver local --opt type=nfs --opt o=addr=192.168.100.10,rw --opt device=:/myvolume/data myvol5`

* `sudo docker inspect myvol5`
```
[
    {
        "CreatedAt": "2022-09-15T18:37:28+04:30",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/myvol5/_data",
        "Name": "myvol5",
        "Options": {
            "device": ":/myvolume/data",
            "o": "addr=192.168.100.10,rw",
            "type": "nfs"
        },
        "Scope": "local"
    }
]
```

* `docker volume rm myvol3` if is not in used. (up or exit means in use)

* `docker volume prune` : delete all unused volumes
```
WARNING! This will remove all local volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Volumes:
myvol
myvol2
myvol3

Total reclaimed space: 0B
```

### 2. bind mount

An approach to store some data of the container outside of the container. (In the disk.)

* `docker run -itd --name mycent --mount type=volume,source=myvol,target=/data centos:latest /bin/bash` == like `-v ...`

* `docker run -itd --name mycent --read-only --mount type=bind,source=/mnt,target=/mnt centos:latest`

**All paths are read-only except mount paths**

### 3. tmpfs

An approach to store some data of the container outside of the container. (In memory: ram)

* to put in memory
* secret
* temporary (after each stop/start the data will be removed)

* `docker run -itd --name mycent --tmpfs /tmp centos`

* `docker run -itd --name mycent --mount type=tmpfs,destination=/tmp centos`

---

## Docker Network

* `Container Network Model` - `CNM`    
   Design spec (no implementeation)
   - Sandboxes: isolated **network stack**, ethernet interfaces, ports, routing tables, dns and ...
   - Endpoints: are virtual network interface like normal network interface for making connections.
   - Networks: a software implementation of an 802.1d bridge (second layer switch = like a physical switch but virtual)

* `libnetwork` (control and management)
   network spec Implemented by GO

* `Drivers` (data)
  - bridge (single-host): default, for standalone container
  - host: for standalone container - **remove network isolation**
  - none or null
  - overlay
  - macvlan

* `docker network ls`
```bash
NETWORK ID     NAME      DRIVER    SCOPE
6b541beb5be4   bridge    bridge    local
10b5ace57d61   host      host      local
e7890d326fa5   none      null      local
```

* `docker network ls -f driver=bridge`

* `docker network inspect bridge`

* `docker inspect container` # check the NetworkSettings

* `docker network create -d <driver> name`          
  `docker network create -d bridge mynet`           
  `docker network create mynet`

  `docker network ls`
```bash
NETWORK ID     NAME       DRIVER    SCOPE
6b541beb5be4   bridge     bridge    local  --> system bridge
10b5ace57d61   host       host      local
bf4f94333d00   localnet   bridge    local  --> user defined bridge
e7890d326fa5   none       null      local
```

* `docker run -it --name cent4 --network localnet centos`

* **In user defined bridge you have name resulotion but in the system bridge no**(`ping cent1`)   
* **User defined bridges provide better isolation** since the system bridge is default.   
* **User defined bridges are better configurable**

* `docker network connect localnet cent1`
* `docker network connect --ip 192.168.100.50 localnet cent1`

* `docker network disconnect localnet cent1`

* `docker network create --subnet 192.168.100.0/24 mynet`

* `docker network create --subnet 192.168.100.0/24 --gateway 192.168.100.10 mynet1`

* `docker run -itd --name cent2 --network mynet --ip 192.168.100.10 centos` # set static ip on container

* `docker run -it --network host centos`

### Port Mappings (Publishing)

* `docker run -itd --name myhttp -p 5000:80 httpd`

* `# docker port myhttp`
```
80/tcp -> 0.0.0.0:5000
80/tcp -> :::5000
```

**docker automatically add rules to iptable. if not run these:**
* sysctl net.ipv4.conf.all.forwarding=1
* sudo iptables -P FORWARD ACCEPT

---

## Troubleshooting

* `docker run -it --net container:<container_name> nicolaka/netshoot`

* `docker run -it --net host nicolaka/netshoot` # host network troubleshooting

**netshoot** is a powerful networking tshooting tools.

Example:

1. `docker run -itd --name mynginx -p 80:80 nginx`
2. `docker run -it --network container:mynginx nicolaka/netshoot`

```bash
                    dP            dP                           dP   
                    88            88                           88   
88d888b. .d8888b. d8888P .d8888b. 88d888b. .d8888b. .d8888b. d8888P 
88'  `88 88ooood8   88   Y8ooooo. 88'  `88 88'  `88 88'  `88   88   
88    88 88.  ...   88         88 88    88 88.  .88 88.  .88   88   
dP    dP `88888P'   dP   `88888P' dP    dP `88888P' `88888P'   dP   
                                                                    
Welcome to Netshoot! (github.com/nicolaka/netshoot)
                                                                


 0e8c6a8befd1  ~  

```

Example 2:
1. `docker run -itd --name perf-test-a --network perf-test nicolaka/netshoot iperf -s -p 9999`
2. `docker run -itd --name perf-test-b --network perf-test nicolaka/netshoot iperf -c perf-test-a -p 9999`
3. `docker logs perf-test-a`
```bash
------------------------------------------------------------
Server listening on TCP port 9999
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  1] local 172.18.0.2 port 9999 connected with 172.18.0.3 port 44978
[ ID] Interval       Transfer     Bandwidth
[  1] 0.00-10.00 sec  44.1 GBytes  37.9 Gbits/sec
```

Example 3: (`nmap`)

1. `docker run -itd --name mynginx -p 80:80 nginx`
2. `docker run -it --privileged nicolaka/netshoot nmap -p 1-1024 -dd 172.17.0.2 | grep open`
```bash
Discovered open port 80/tcp on 172.17.0.2
80/tcp   open   http                syn-ack ttl 64
260/tcp  closed openport            reset ttl 64
557/tcp  closed openvms-sysipc      reset ttl 64
```

**--privileged**: root access from this container to docker host. (**SECURITY ISSUE**)

Example 4: (`iftop`)

1. `docker run -itd --name mynginx -p 80:80 nginx`
2. `docker run -it --network container:mynginx nicolaka/netshoot iftop -i eth0`

Example 5: (`ctop`)

1. `docker run -it --name cent1 centos` --> CTRL+D
2. `docker run -it --name cent2 centos` --> CTRL+D
3. `docker run -itd --name cent3 centos`
4. `docker run -itd --name cent4 centos`
5. `docker run -it -v /var/run/docker.sock:/var/run/docker.sock nicolaka/netshoot ctop`

```bash
  ctop - 19:15:31 UTC   5 containers                                                                                                                                                 

     NAME                           CID          CPU                      MEM                      NET RX/TX                IO R/W                   PIDS UPTIME

   ⏵ busy_faraday                   bfdb310a0f2a             0%                   12M / 12G        90B / 437B               0B / 0B                  13   0s                      
   ⏵ cent3                          46333bb58b44             0%                   920K / 12G       3K / 437B                0B / 0B                  1    44s
   ⏵ cent4                          2852b96dfcfb             0%                   936K / 12G       1K / 523B                0B / 0B                  1    38s
   ⏹ cent1                          9acea9386209             -                        -            -                        -                        -    6s
   ⏹ cent2                          fc426d40cb96             -                        -            -                        -                        -    0s
   ```

**-v** to access to the docker host service to see the container informations for using them by `ctop`.


Example 5: (`termshark`)

1. `docker run -itd --name mynginx -p 80:80 nginx`
2. `docker run --rm --cap-add NET_ADMIN --cap-add CAP_NET_RAW -it --network  container:mynginx nicolaka/netshoot termshark -i eth0 tcp`


Example 6: (`nc` - check firewall issue)

1. `docker network create mynet`
2. `docker run -itd --name mynginx -p 80:80 --network mynet nginx`
3. `docker run -itd --name mynetcat --network mynet nicolaka/netshoot nc -vz mynginx 80`