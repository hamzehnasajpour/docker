# My Docker notes


## Docker Logs



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


### 2. bind mount

An approach to store some data of the container outside of the container. (In the disk.)



### 3. tmpfs

An approach to store some data of the container outside of the container. (In memory: ram)



### 4. Commands

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

* `docker run -it --name cent6 -v myvol:/data -v myvol2:/data2 centos`

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