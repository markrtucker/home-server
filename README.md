# Home Server

This repo captures notes on setting up my home Ubuntu server. I may add scripts and/or other automation here in the future. For now, this is installed on an old PC - I'll likely want to upgrade the hardware and will need to reinstall a bunch of this stuff, so this is mostly just for my own reference.

## Basic Installation

I installed Ubuntu 20.4 LTS because I was looking for a stable OS to run kubernetes. The choice of 20.04 over 18.04 was fairly arbitrary, though the inclusion of only Python 3 in 20.04 was a factor.

## Post Installation Set Up

This is a recorded (partial) history of basic post-install commands:

Install latest updates
```
sudo apt-get upgrade
```


I did install docker, though I'm not sure if this is strictly necessary.
```
sudo apt install docker.io
sudo groupadd docker
sudo usermod -aG docker $USER
docker run hello-world
sudo systemctl enable docker
```

Next, I installed microk8s.
```
sudo snap install microk8s --classic
sudo usermod -aG microk8s mtucker
sudo chown -f -R $USER ~/.kube
microk8s status
microk8s enable dns dashboard ingress istio prometheus
```

## Bash customization

I've done minimal shell customization so far. Just this
```
$ cat ~/.bash_aliases
alias kc='microk8s kubectl'
```

TODO: Set up zsh?

## Future work

I have a long to-do list. Here's a quick summary
- [ ] Figure out microk8s ingress
- [ ] Set up microk8s dashboard - visible on home network
- [ ] Install FluxCD to auto-deploy stuff to microk8s
- [ ] Deploy PiHole (5.0?) docker container to microk8s
   - [ ] Reconfigure home network to turn off existing PiHole
   
