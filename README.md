# exposed_minikube


## Increase sysctl.fs.inotify.max_user_watch
```
sudo vim /etc/sysctl.conf 
```

```
fs.inotify.max_user_watches = 524288
fs.inotify.max_user_instances = 512
```
## Configure ZSH Arc size

```
sudo vim /etc/modprobe.d/zfs.conf
```

```
# Setting up ZFS ARC size on Ubuntu as per our needs
# Set Max ARC size => 2GB == 2147483648 Bytes
options zfs zfs_arc_max=2147483648
 
# Set Min ARC size => 1GB == 1073741824
options zfs zfs_arc_min=1073741824
```

```
sudo update-initramfs -u -k all
```

REBOOT THE COMPUTER NOW

## Start minikube
```
minikube start \
 --kubernetes-version=v1.20.0 \
 --nodes=6 \
 --driver=docker \
 --addons=ingress
```

## Open ports

```
# Get minikube IP
MINIKUBE_IP=$(minikube ip)

# Get minikube network interface
MINIKUBE_INTERFACE=$(ls /sys/class/net/ |grep br) -o $(ls /sys/class/net/ |grep br)

# Get Ingress http nodePort
HTTP_PORT=$(minikube kubectl -- -n ingress-nginx get svc/ingress-nginx-controller -ojson  | jq -r '.spec.ports[0].nodePort')
# Get Ingress https nodePort
HTTPS_PORT=$(minikube kubectl -- -n ingress-nginx get svc/ingress-nginx-controller -ojson  | jq -r '.spec.ports[1].nodePort')

# Expose Docker Ingress ports
sudo iptables -A DOCKER -d ${MINIKUBE_IP}/32 ! -i ${MINIKUBE_INTERFACE} -o ${MINIKUBE_INTERFACE} -p tcp -m tcp --dport ${HTTP_PORT} -j ACCEPT
sudo iptables -A DOCKER -d ${MINIKUBE_IP}/32 ! -i ${MINIKUBE_INTERFACE} -o ${MINIKUBE_INTERFACE} -p tcp -m tcp --dport ${HTTPS_PORT} -j ACCEPT

# Forward host to docker
sudo iptables -t nat -A PREROUTING -i enp5s0 -p tcp --dport 80 -j DNAT \
      --to ${MINIKUBE_IP}:${HTTP_PORT}

sudo iptables -t nat -A PREROUTING -i enp5s0 -p tcp --dport 443 -j DNAT \
      --to ${MINIKUBE_IP}:${HTTPS_PORT}

sudo iptables -t nat -A PREROUTING -i enp5s0 -p tcp --dport 8443 -j DNAT \
      --to ${MINIKUBE_IP}:8443
```
