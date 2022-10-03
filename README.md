Easily redirect (ssh) users to podman/docker containers using a tiny shellscript:

```shell
server # wget https://raw.githubusercontent.com/coderofsalvation/podmanr/main/podmanr -O /usr/bin/podmanr
server # chmod +x /usr/bin/podmanr
server # podmanr adduser john
server # grep john /etc/passwd
john:x:1000:1000:human:/home/john:/usr/bin/podmanr
```

> PROFIT!

## How does it work?

<img src="https://raw.githubusercontent.com/coderofsalvation/podmanr/main/.dtp/diagram.gif"/>

> basically: install podman and set the `podmanr` shellscript as the user shell

Let's take a look at what the (highly hackable) shellscript boils down to:

```
image=alpine:3.16         # default image for users without Dockerfile in homedir
shell(){   ... }          # the actual usershell
build(){   ... }          # builds user $HOME/Dockerfile (if present)
init(){    ... }          
adduser(){ ... }          # lets root add users
```

#### Full demo 

```shell
server # wget https://raw.githubusercontent.com/coderofsalvation/podmanr/main/podmanr -O /usr/bin/podmanr
server # chmod +x /usr/bin/podmanr
server # podmanr adduser john
server # grep john /etc/passwd
john:x:1000:1000:human:/home/john:/usr/bin/podmanr
server # hostname
server

laptop $ ssh john@server
/ # hostname
jfe2jfe3ac87298                  <---- we're in default container!

/ # echo "FROM alpine:3.16" > Dockerfile
/ # exit

laptop $ ssh john@server
/ # hostname
abe68768baeb8                    <---- our own custom container!
```

## Share/Expose images to users

Normally rootless containers are based on images pulled by the user, however podmanr-users can list images 
from an additional (shared) store:

```shell 
# podman --root /var/lib/containers pull debian:bookworm 
# chmod -R a+rx /var/lib/containers
# su -l -c /bin/sh myuser             # fake ssh login to container
$ podman images
REPOSITORY                TAG         IMAGE ID      CREATED      SIZE        R/O
docker.io/library/debian  woodwork    9c6f07244728  7 weeks ago  5.83 MB     true
```

## Customization ideas 

* freeze Dockerfile: `chown root:root /home/someuser/Dockerfile && chmod 744 /home/someuser/Dockerfile`
* apply limits & introduce extra settings:  
  * create `/home/username/root`
  * modify podmanr to launch `podman run` with `-v /home/$(whoami)/root:/root` and/or `--cpus = $(cat .cpus)` e.g. 
  * this uses (and hides) files/configuration-files in `/home/username/*` for the user (which operates inside `/home/username/root`)

## Troubleshooting

```
# su -l  myuser             # fake ssh login to container
$                           # to trigger Dockerfile build e.g.

# su -l -s /bin/sh myuser   # jump into normal shell
$                           # to inspect, run 'podman ps' 'podman images' e.g.

```
