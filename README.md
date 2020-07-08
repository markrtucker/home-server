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

## Windows Client Setup

I set up my window client to use kubectl to control the cluster remotely. (Ideally, I will do very little work directly on the server.)

There was a fair bit of trial-and-error here, especially around TLS and getting the right cert trusted in the right way. Here's what I *think* is necessary.

### Cluster configuration

Get the cluster configuration from the server

```
$ kc config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:16443
  name: microk8s-cluster
contexts:
- context:
    cluster: microk8s-cluster
    user: admin
  name: microk8s
current-context: microk8s
kind: Config
preferences: {}
users:
- name: admin
  user:
    token: Tm9uWW....
```

Copy this information into a config file locally, making a few edits:
1) Change the IP address from `127.0.0.1` to the server's IP address
2) Replace `DATA+OMITTED` with the base64 encoded certificate

To get the base64 encoded certificate, run this on the server:
```
$ cat /var/snap/microk8s/current/certs/ca.crt | base64
```

This trusts the "root" certificate used to sign the various other certs used by the k8s cluster.

With these changes, the file on Windows looks like this:
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0t......long string omitted...tLS0tLQ==
    server: https://192.168.1.87:16443
  name: microk8s-cluster
contexts:
- context:
    cluster: microk8s-cluster
    user: admin
  name: microk8s
current-context: microk8s
kind: Config
preferences: {}
users:
- name: admin
  user:
    token: Tm9uWW....
```

I then set the `KUBECONFIG` environment variable to point at this file.

With that, I have kubectl working on Windows and talking to my cluster:

```
C:\Users\Mark\k8s>kubectl cluster-info
Kubernetes master is running at https://192.168.1.87:16443
CoreDNS is running at https://192.168.1.87:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://192.168.1.87:16443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.



C:\Users\Mark\k8s>kubectl get pods --namespace kube-system
NAME                                        READY   STATUS    RESTARTS   AGE
coredns-588fd544bf-nnhqx                    1/1     Running   0          6d22h
dashboard-metrics-scraper-59f5574d4-66x6f   1/1     Running   0          6d22h
kubernetes-dashboard-6d97855997-6zrhr       1/1     Running   0          6d22h
metrics-server-c65c9d66-jkdpq               1/1     Running   0          6d22h
```


## Future work

I have a long to-do list. Here's a quick summary
- [ ] set up zsh??
- [ ] create 'dotfiles' repo to save shell customization
- [ ] Figure out microk8s ingress
- [ ] Set up microk8s dashboard - visible on home network
- [ ] Install FluxCD to auto-deploy stuff to microk8s
- [ ] Deploy PiHole (5.0?) docker container to microk8s
   - [ ] Reconfigure home network to turn off existing PiHole
   
