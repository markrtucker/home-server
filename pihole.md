# Installing Pi-hole

Pi-hole is a DNS server that acts as a whole network ad-blocker. It has a block-list of known ad servers that is resolves to fake addresses such that ads are never loaded by the browser.

I've been running pi-hole at home for a while on a Raspberry Pi. With this project, I'm installing Pi-hole on my server and having it hosted by microk8s.


## Preparation

Per the [documentation](https://github.com/pi-hole/docker-pi-hole/#installing-on-ubuntu), there is an existing service on Ubuntu that listens on port 53. This needs to be disabled before installing Pi-hole.

```
sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf
sudo sh -c 'rm /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf'
systemctl restart systemd-resolved
```
