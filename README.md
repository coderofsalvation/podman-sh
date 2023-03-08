Easily redirect (ssh) users to podman/docker containers using a tiny shellscript:

```shell
server # wget https://raw.githubusercontent.com/coderofsalvation/podman-sh/main/podman-sh -O /usr/bin/podman-sh
server # chmod +x /usr/bin/podman-sh
server # podman-sh adduser john
server # grep john /etc/passwd
john:x:1000:1000:human:/home/john:/usr/bin/podman-sh
```

> PROFIT!

## How does it work?

> NOTE: `podmanr` has been renamed to `podman-sh` (thx for the suggestion [@rhatdan](https://twitter.com/rhatdan))

<img src="https://raw.githubusercontent.com/coderofsalvation/podman-sh/main/.dtp/diagram.gif"/>

> basically: install podman and set `podman-sh` as the user shell

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
server # wget https://raw.githubusercontent.com/coderofsalvation/podman-sh/main/podman-sh -O /usr/bin/podman-sh
server # chmod +x /usr/bin/podman-sh
server # podman-sh adduser john
server # grep john /etc/passwd
john:x:1000:1000:human:/home/john:/usr/bin/podman-sh
server # hostname
server

laptop $ ssh john@server
/ # hostname
jfe2jfe3ac87298                           # <-- we're in default container!

/ # echo "FROM alpine:3.16" > Dockerfile  # update file to trigger build
/ # exit

laptop $ ssh john@server
/ # hostname
abe68768baeb8                              # <-- our own custom container!
```

## Share/Expose images to users

Normally rootless containers are based on images pulled by the user, however podman-sh-users can list images 
from an additional (shared) store:

```shell 
# podman --root /var/lib/containers pull debian:bookworm 
# chmod -R a+rx /var/lib/containers
# su -l -c /bin/sh myuser             # fake ssh login to container
$ podman images
REPOSITORY                TAG         IMAGE ID      CREATED      SIZE        R/O
docker.io/library/debian  woodwork    9c6f07244728  7 weeks ago  5.83 MB     true
```

> NOTE: you can also run `podman-sh pull <the_image>` which is an alias for the above cmds

## Customization ideas 

* put a `.boot` shellscript in a homedir to override the `podman run` command when logging in
* freeze Dockerfile: `chown root:root /home/someuser/Dockerfile && chmod 744 /home/someuser/Dockerfile`
* apply limits & introduce extra settings:  
  * create `/home/username/root`
  * modify podman-sh to launch `podman run` with `-v /home/$(whoami)/root:/root` and/or `--cpus = $(cat .cpus)` e.g. 
  * this uses (and hides) files/configuration-files in `/home/username/*` for the user (which operates inside `/home/username/root`)

## Troubleshooting

```
# su -l  myuser             # fake ssh login to container
$                           # to trigger Dockerfile build e.g.

# su -l -s /bin/sh myuser   # jump into normal shell
$                           # to inspect, run 'podman ps' 'podman images' e.g.

```

> NOTE: Rootless podman can be tricky to setup (configuring overlay e.g.), here are some [installation notes](.alpine-notes.txt) for alpine linux on a VPS.

