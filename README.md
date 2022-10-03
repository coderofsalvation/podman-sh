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

> Bonus: images in `/var/lib/containers` are shared. Which means that root can make/pull images and make them available to users (which see them in the list of available images when `Dockerfile` is being build (or run `cat ~/.images`))

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

## Customization ideas 

* freeze Dockerfile: `chown root:root /home/someuser/Dockerfile && chmod 744 /home/someuser/Dockerfile`
* apply limits & introduce extra settings:  
  * create `/home/username/root`
  * modify podmanr to launch `podman run` with `-v /home/$(whoami)/root:/root` and/or `--cpus = $(cat .cpus)` e.g. 
  * this uses (and hides) files/configuration-files in `/home/username/*` for the user (which operates inside `/home/username/root`)

## Troubleshooting

```
# su -l -s /bin/sh myuser                # simulate ssh login to container
$                                        # to trigger Dockerfile build e.g.

# su -l -s /bin/sh myuser                # jump into normal shell
$

```
