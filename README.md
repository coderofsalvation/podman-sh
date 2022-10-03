Easily redirect regular *nix users to podman containers using a tiny shellscript:

```shell
server # wget https://raw.githubusercontent.com/coderofsalvation/podmanr/main/podmanr -O /usr/bin/podmanr
server # chmod +x /usr/bin/podmanr
server # podmanr adduser john
```

> PROFIT!

## How does it work?

> basically: install podman and set the `podmanr` shellscript as the user shell

```shell
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

## Explanation

Let's take a look at the (highly hackable) shellscript:

```
image=alpine:3.16         # default image for users without Dockerfile in homedir
shell(){   ... }          # the actual usershell
build(){   ... }          # builds user $HOME/Dockerfile (if present)
init(){    ... }          
adduser(){ ... }          # lets root add users
```

<img src=".dtp/diagram.jpg"/>
