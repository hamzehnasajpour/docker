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

* `docker volume creat myvol`
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

