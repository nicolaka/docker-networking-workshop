
# Dockercon 2017 Networking Workshop
## Demo Commands

### Show basic concepts of host mode networking

```
$ ip addr show

$ ip route 

$ docker network ls

$ docker network inspect host

$ docker run --rm -it --net host nginx 

# ip addr show

# ip route 

# curl localhost

$ netstat -plt

$ curl localhost

$ docker run --rm -it --net host centos sh

#  curl localhost

$ docker run --rm -it --net host nginx
```

### Demonstrate end to end packet flow of host mode network

```
Host1 $ docker run -d --rm --net host chrch/netshoot netgen 10.0.18.212 5000

Host2 $ docker run -d -it --net host chrch/netshoot netgen 10.0.7.28 5000

$ sudo docker run --privileged --net=host -v /:/rootfs:ro fntlnz/netcan -rootfs=/rootfs

$ sudo tcpdump -vvvn -i eth0 port 5000
```

### Demonstrate port mapping

```
$ docker run -d -p 8000:80 nginx

$ curl localhost:8000

$ iptables -t nat -S | grep 8000
```

### Explore bridge driver and namespace fundamentals
```
$ docker network ls                   << This is the default bridge network. Show the students the bridge network with NAME: bridge, DRIVER: bridge, and SCOPE: local

$ docker inspect bridge

$ brctl show   (will have to have bridge-utils installed)

$ docker network create -d bridge br

$ docker run -it --net br alpine sh

# ip add show

# ip route

# curl localhost  (can’t connect to nginx container that is running in net=host mode)

CTRL P-Q

$ docker inspect $(docker ps -lq)

$ ls /var/run/docker/netns

$ brctl show

$ ip addr show

$ docker rm -f $(docker ps -aq)
```


### Inspect traffic between containers on same host

```
$ docker run -d --rm --net br --name c1 chrch/netshoot netgen c2 5000

$ docker run -it --rm --net br --name c2 chrch/netshoot netgen c1 5000

$ tcpdump -nXs 0 -i eth0 port 5000

Grab the veth interface of one of c1
$ docker exec -it c1 ip add show eth0

Find corresponding veth in the host network namespace
$ tcpdump -nXs 0 -i veth18265bc port 5000
```

### Inspect traffic between containers on different 
```
Host1 $ docker run -d --rm -p 5000:5000 chrch/netshoot netgen ip-172-31-18-122 5000

Host2 $ docker run -d --it -p 5000:5000 chrch/netshoot netgen ip-172-31-21-237 5000

$  tcpdump -nXs 0 -i eth0 port 5000
```



