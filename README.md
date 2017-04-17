# Docker Networking Workshop

Hi, welcome to the Docker Networking Workshop!

You will get your hands dirty by going through examples of a few basic networking concepts, learn about Bridge and Overlay networking, and finally learning about the Swarm Routing Mesh.

> **Difficulty**: Beginner to Intermediate

> **Time**: Approximately 45 minutes

> **Tasks**:
>
> * [Prerequisites](#prerequisites)
> * [Section #1 - Networking Basics](#task1)
> * [Section #2 - Bridge Networking](#task2)
> * [Section #3 - Overlay Networking](#task3)

## Document conventions

When you encounter a phrase in between `<` and `>`  you are meant to substitute in a different value. 

For instance if you see `ssh <username>@<hostname>` you would actually type something like `ssh ubuntu@node0.ivaf2i2atqouppoxund0tvddsa.jx.internal.cloudapp.net`

You will be asked to SSH into various nodes. These nodes are referred to as **node0** or **node1**. These tags correspond to the very beginning of the hostnames found on the hands on labs welcome card you were given. 

## <a name="prerequisites"></a>Prerequisites

This lab requires two Linux nodes with Docker 17.03 (or higher) installed.

Also, please make sure you can SSH into the Linux nodes. If you haven't already done so, please SSH in to **node0** and **node1**.

```
$ ssh ubuntu@<node0 IP address>
```

and 

```
$ ssh ubuntu@<node1 IP address>
```

**Note: Password will be provided during the workshop.**

# <a name="task1"></a>Section #1 - Networking Basics

## <a name="list_networks"></a>Step 1: The Docker Network Command

If you haven't already done so, please SSH in to **node0**.

```
$ ssh ubuntu@<node0 IP address>
```

The `docker network` command is the main command for configuring and managing container networks. Run the `docker network` command from **node0**.

```
$ docker network

Usage:    docker network COMMAND

Manage networks

Options:
      --help   Print usage

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
```

The command output shows how to use the command as well as all of the `docker network` sub-commands. As you can see from the output, the `docker network` command allows you to create new networks, list existing networks, inspect networks, and remove networks. It also allows you to connect and disconnect containers from networks.

## <a name="list_networks"></a>Step 2: List networks

Run a `docker network ls` command on **node0** to view existing container networks on the current Docker host.

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
3430ad6f20bf        bridge              bridge              local
a7449465c379        host                host                local
06c349b9cc77        none                null                local
```

The output above shows the container networks that are created as part of a standard installation of Docker.

New networks that you create will also show up in the output of the `docker network ls` command.

You can see that each network gets a unique `ID` and `NAME`. Each network is also associated with a single driver. Notice that the "bridge" network and the "host" network have the same name as their respective drivers.

## <a name="inspect"></a>Step 3: Inspect a network

The `docker network inspect` command is used to view network configuration details. These details include; name, ID, driver, IPAM driver, subnet info, connected containers, and more.

Use `docker network inspect <network>` on **node0** to view configuration details of the container networks on your Docker host. The command below shows the details of the network called `bridge`.

```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "3430ad6f20bf1486df2e5f64ddc93cc4ff95d81f59b6baea8a510ad500df2e57",
        "Created": "2017-04-03T16:49:58.6536278Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

> **NOTE:** The syntax of the `docker network inspect` command is `docker network inspect <network>`, where `<network>` can be either network name or network ID. In the example above we are showing the configuration details for the network called "bridge". Do not confuse this with the "bridge" driver.


## <a name="list_drivers"></a>Step 4: List network driver plugins

The `docker info` command shows a lot of interesting information about a Docker installation.

Run the `docker info` command on **node0** and locate the list of network plugins.

```
$ docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.03.1-ee-3
Storage Driver: aufs
<Snip>
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
Swarm: inactive
Runtimes: runc
<Snip>
```

The output above shows the **bridge**, **host**,**macvlan**, **null**, and **overlay** drivers.

# <a name="task2"></a>Section #2 - Bridge Networking


## <a name="connect-container"></a>Step 1: The Basics

Every clean installation of Docker comes with a pre-built network called **bridge**. Verify this with the `docker network ls` command on **node0**.

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
3430ad6f20bf        bridge              bridge              local
a7449465c379        host                host                local
06c349b9cc77        none                null                local
```

The output above shows that the **bridge** network is associated with the *bridge* driver. It's important to note that the network and the driver are connected, but they are not the same. In this example the network and the driver have the same name - but they are not the same thing!

The output above also shows that the **bridge** network is scoped locally. This means that the network only exists on this Docker host. This is true of all networks using the *bridge* driver - the *bridge* driver provides single-host networking.

All networks created with the *bridge* driver are based on a Linux bridge (a.k.a. a virtual switch).


## <a name="connect-container"></a>Step 2: Connect a container

The **bridge** network is the default network for new containers. This means that unless you specify a different network, all new containers will be connected to the **bridge** network.

Create a new bridge network on **node0** and call it `br`.

```
$ docker network create -d bridge br
846af8479944d406843c90a39cba68373c619d1feaa932719260a5f5afddbf71
```

Now create a container called `c1` and attach it to your new `br` network.

```
$ docker run -itd --net br --name c1 alpine sh
846af8479944d406843c90a39cba68373c619d1feaa932719260a5f5afddbf71
```

This command will create a new container based on the `alpine:latest` image. 

Running `docker network inspect bridge` will show the containers on that network.

```
$ docker network inspect br

[
    {
        "Name": "br",
        "Id": "e7b30cacc686ff891a5a5ea393e055c309a07bc652feed375821e2f78faf9aa0",
        "Created": "2017-04-13T13:19:37.611068665Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "086c4c0cc18c7f603279b728d9baac4b63d25941f576b37c5d1e988de6202410": {
                "Name": "c1",
                "EndpointID": "06eac477e1a04f2c5e676633ddab344086104511470c539a2fb7aedf8b1d58f8",
                "MacAddress": "02:42:ac:11:00:06",
                "IPv4Address": "172.17.0.6/16",
                "IPv6Address": ""
```



## <a name="ping_local"></a>Step 3: Test network connectivity

The output to the previous `docker network inspect` command shows the IP address of the new container. In the previous example it is "172.17.0.6" but yours might be different.

Ping the IP address of the container from the shell prompt of your Docker host by running `ping -c 3 <IPv4 Address>` on **node0**. Remember to use the IP of the container in **your** environment.

You can get the IP address of the container directly from the Docker engine by running `docker inspect --format "{{ .NetworkSettings.Networks.br.IPAddress }}" c1`.

```
$ ping -c 3 172.17.0.6
PING 172.17.0.6 (172.17.0.6) 56(84) bytes of data.
64 bytes from 172.17.0.6: icmp_seq=1 ttl=64 time=0.072 ms
64 bytes from 172.17.0.6: icmp_seq=2 ttl=64 time=0.029 ms
64 bytes from 172.17.0.6: icmp_seq=3 ttl=64 time=0.048 ms
...
```

The replies above show that the Docker host can ping the container over the **bridge** network. But, we can also verify the container can connect to the outside world too. 

Enter in to the `c1` container that you created using the command `docker exec`. We will pass the `sh` command to `docker exec` which puts us in to an interactive shell inside the container.

Enter in to the container and inspect the interfaces of the container

```
$ docker exec -it c1 sh
 # ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
879: eth0@if880: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:13:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.6/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe13:2/64 scope link
       valid_lft forever preferred_lft forever
```

Prove that containers can gain outside access by pinging `www.docker.com`.

```
 # ping -c 3 www.docker.com
PING www.docker.com (104.239.220.248): 56 data bytes
64 bytes from 104.239.220.248: seq=0 ttl=36 time=77.722 ms
64 bytes from 104.239.220.248: seq=1 ttl=36 time=77.865 ms
64 bytes from 104.239.220.248: seq=2 ttl=36 time=77.830 ms
```

Exit out of the container.

```
 # exit
```

Now you will create a second container on this bridge so you can test connectivity between them.

```
$ docker run -itd --net br --name c2 alpine sh
75f840c9d17b2921c1e78555c97cd5116e1563b1e33f9328bd5b0a8e1c55b520
```

Enter the `c2` container with `docker exec` and try to ping the IP address of `c1`.

```
$ docker exec -it c2 sh
 # ping -c 3 172.17.0.6
PING 172.17.0.6 (172.17.0.6): 56 data bytes
64 bytes from 172.17.0.6: seq=0 ttl=64 time=0.091 ms
64 bytes from 172.17.0.6: seq=1 ttl=64 time=0.077 ms
64 bytes from 172.17.0.6: seq=2 ttl=64 time=0.079 ms
```

Now ping container `c1` using it's name. The Docker engine will provide the resolution automatically for all container names and service names.


```
 # ping -c 3 c1
PING c1 (172.17.0.6): 56 data bytes
64 bytes from 172.17.0.6: seq=0 ttl=64 time=0.091 ms
64 bytes from 172.17.0.6: seq=1 ttl=64 time=0.077 ms
64 bytes from 172.17.0.6: seq=2 ttl=64 time=0.079 ms
```

Exit container `c2` and remove these two containers from this host.

```
# exit
$ docker rm -f $(docker ps -aq)
```



## <a name="nat"></a>Step 4: Configure NAT for external connectivity

In this step we'll start a new **NGINX** container and map port 8080 on the Docker host to port 80 inside of the container. This means that traffic that hits the Docker host on port 8080 will be passed on to port 80 inside the container.

> **NOTE:** If you start a new container from the official NGINX image without specifying a command to run, the container will run a basic web server on port 80.

Start a new container based off the official NGINX image by running `docker run --name web1 -d -p 8080:80 nginx` on **node0**.

```
$ docker run --name web1 -d -p 8080:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
6d827a3ef358: Pull complete
b556b18c7952: Pull complete
03558b976e24: Pull complete
9abee7e1ef9d: Pull complete
Digest: sha256:52f84ace6ea43f2f58937e5f9fc562e99ad6876e82b99d171916c1ece587c188
Status: Downloaded newer image for nginx:latest
4e0da45b0f169f18b0e1ee9bf779500cb0f756402c0a0821d55565f162741b3e
```

Review the container status and port mappings by running `docker ps` on **node0**.

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                           NAMES
4e0da45b0f16        nginx               "nginx -g 'daemon ..."   2 minutes ago       Up 2 minutes        443/tcp, 0.0.0.0:8080->80/tcp   web1
```

The top line shows the new **web1** container running NGINX. Take note of the command the container is running as well as the port mapping - `0.0.0.0:8080->80/tcp` maps port 8080 on all host interfaces to port 80 inside the **web1** container. This port mapping is what effectively makes the containers web service accessible from external sources (via the Docker hosts IP address on port 8080).

Now that the container is running and mapped to a port on a host interface you can test connectivity to the NGINX web server.

To complete the following task you will need the IP address of your Docker host. This will need to be an IP address that you can reach (e.g. your lab is hosted in Azure so this will be the instance's Public IP - the one you SSH'd into). Just point your web browser to the IP and port 8080 of your Docker host. Also, if you try connecting to the same IP address on a different port number it will fail.

If for some reason you cannot open a session from a web broswer, you can connect from your Docker host using the `curl 127.0.0.1:8080` command on **node0**.

```
$ curl 127.0.0.1:8080
<!DOCTYPE html>
<html>
<Snip>
<head>
<title>Welcome to nginx!</title>
    <Snip>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

If you try and curl the IP address on a different port number it will fail.

> **NOTE:** The port mapping is actually port address translation (PAT).

# <a name="task3"></a>Section #3 - Overlay Networking

## <a name="connect-container"></a>Step 1: The Basics

In this step you'll initialize a new Swarm, join a single worker node, and verify the operations worked.

Run `docker swarm init` on **node0**.

```
$ docker swarm init
Swarm initialized: current node (rzyy572arjko2w0j82zvjkc6u) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-69b2x1u2wtjdmot0oqxjw1r2d27f0lbmhfxhvj83chln1l6es5-37ykdpul0vylenefe2439cqpf \
    10.0.0.5:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

If you haven't already done so, please SSH in to **node1**.

```
$ ssh ubuntu@<node0 IP address>
```

Copy the entire `docker swarm join ...` command that is displayed as part of the output from your terminal output on **node0**. Then, paste the copied command into the terminal of **node1**.

```
$ docker swarm join \
>     --token SWMTKN-1-69b2x1u2wtjdmot0oqxjw1r2d27f0lbmhfxhvj83chln1l6es5-37ykdpul0vylenefe2439cqpf \
>     10.0.0.5:2377
This node joined a swarm as a worker.
```

Run a `docker node ls` on **node0** to verify that both nodes are part of the Swarm.

```
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
ijjmqthkdya65h9rjzyngdn48    node1   Ready   Active
rzyy572arjko2w0j82zvjkc6u *  node0   Ready   Active        Leader
```

The `ID` and `HOSTNAME` values may be different in your lab. The important thing to check is that both nodes have joined the Swarm and are *ready* and *active*.

## <a name="create_network"></a>Step 2: Create an overlay network

Now that you have a Swarm initialized it's time to create an **overlay** network.

Create a new overlay network called "overnet" by running `docker network create -d overlay overnet` on **node0**.

```
$ docker network create -d overlay --subnet 10.10.10.0/24 overnet
wlqnvajmmzskn84bqbdi1ytuy
```

Use the `docker network ls` command to verify the network was created successfully.

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
3430ad6f20bf        bridge              bridge              local
a4d584350f09        docker_gwbridge     bridge              local
a7449465c379        host                host                local
8hq1n8nak54x        ingress             overlay             swarm
06c349b9cc77        none                null                local
wlqnvajmmzsk        overnet             overlay             swarm
```

The new "overnet" network is shown on the last line of the output above. Notice how it is associated with the **overlay** driver and is scoped to the entire Swarm.

> **NOTE:** The other new networks (ingress and docker_gwbridge) were created automatically when the Swarm cluster was created.

Run the same `docker network ls` command from **node1**

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
55f10b3fb8ed        bridge              bridge              local
b7b30433a639        docker_gwbridge     bridge              local
a7449465c379        host                host                local
8hq1n8nak54x        ingress             overlay             swarm
06c349b9cc77        none                null                local
```

Notice that the "overnet" network does **not** appear in the list. This is because Docker only extends overlay networks to hosts when they are needed. This is usually when a host runs a task from a service that is created on the network. We will see this shortly.

Use the `docker network inspect <network>` command to view more detailed information about the "overnet" network. You will need to run this command from **node0**.

```
$ docker network inspect overnet
[
    {
        "Name": "overnet",
        "Id": "wlqnvajmmzskn84bqbdi1ytuy",
        "Created": "0001-01-01T00:00:00Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Attachable": false,
        "Containers": null,
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": null
    }
]
```

## <a name="create_service"></a>Step 3: Create a service

Now that we have a Swarm initialized and an overlay network, it's time to create a service that uses the network.

Execute the following command from **node0** to create a new service called *myservice* on the *overnet* network with two tasks/replicas.

```
$ docker service create --name myservice \
--network overnet \
--replicas 2 \
ubuntu sleep infinity

ov30itv6t2n7axy2goqbfqt5e
```

Verify that the service is created and both replicas are up by running `docker service ls`.

```
$ docker service ls
ID            NAME       MODE        REPLICAS  IMAGE
ov30itv6t2n7  myservice  replicated  2/2       ubuntu:latest
```

The `2/2` in the `REPLICAS` column shows that both tasks in the service are up and running.

Verify that a single task (replica) is running on each of the two nodes in the Swarm by running `docker service ps myservice`.

```
$ docker service ps myservice
ID            NAME         IMAGE          NODE     DESIRED STATE  CURRENT STATE               ERROR  PORTS
riicggj5tuta  myservice.1  ubuntu:latest  node1  Running        Running about a minute ago
nlozn82wsttv  myservice.2  ubuntu:latest  node0  Running        Running about a minute ago
```

The `ID` and `NODE` values might be different in your output. The important thing to note is that each task/replica is running on a different node.

Now that **node1** is running a task on the "overnet" network it will be able to see the "overnet" network. Lets run `docker network ls` from **node1** to verify this.

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
55f10b3fb8ed        bridge              bridge              local
b7b30433a639        docker_gwbridge     bridge              local
a7449465c379        host                host                local
8hq1n8nak54x        ingress             overlay             swarm
06c349b9cc77        none                null                local
wlqnvajmmzsk        overnet             overlay             swarm
```

We can also run `docker network inspect overnet` on **node1** to get more detailed information about the "overnet" network and obtain the IP address of the task running on **node1**.

```
$ docker network inspect overnet
[
    {
        "Name": "overnet",
        "Id": "wlqnvajmmzskn84bqbdi1ytuy",
        "Created": "2017-04-04T09:35:47.526642642Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.10.10.0/24",
                    "Gateway": "10.10.10.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "fbc8bb0834429a68b2ccef25d3c90135dbda41e08b300f07845cb7f082bcdf01": {
                "Name": "myservice.1.riicggj5tutar7h7sgsvqg72r",
                "EndpointID": "8edf83ebce77aed6d0193295c80c6aa7a5b76a08880a166002ecda3a2099bb6c",
                "MacAddress": "02:42:0a:00:00:03",
                "IPv4Address": "10.10.10.3/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": {},
        "Peers": [
            {
                "Name": "node0-f6a6f8e18a9d",
                "IP": "10.10.10.5"
            },
            {
                "Name": "node1-507a763bed93",
                "IP": "10.10.10.6"
            }
        ]
    }
]
```

You should note that as of Docker 1.12, `docker network inspect` only shows containers/tasks running on the local node. This means that `10.10.10.3` is the IPv4 address of the container running on **node1**. Make a note of this IP address for the next step (the IP address in your lab might be different than the one shown here in the lab guide).

## <a name="test"></a>Step 4: Test the network

To complete this step you will need the IP address of the service task running on **node1** that you saw in the previous step (`10.10.10.3`).

Execute the following commands from **node0**.

```
$ docker network inspect overnet
[
    {
        "Name": "overnet",
        "Id": "wlqnvajmmzskn84bqbdi1ytuy",
        "Created": "2017-04-04T09:35:47.362263887Z",
        "Scope": "swarm",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.10.10.0/24",
                    "Gateway": "10.10.10.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "d676496d18f76c34d3b79fbf6573a5672a81d725d7c8704b49b4f797f6426454": {
                "Name": "myservice.2.nlozn82wsttv75cs9vs3ju7vs",
                "EndpointID": "36638a55fcf4ada2989650d0dde193bc2f81e0e9e3c153d3e9d1d85e89a642e6",
                "MacAddress": "02:42:0a:00:00:04",
                "IPv4Address": "10.10.10.4/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.driver.overlay.vxlanid_list": "4097"
        },
        "Labels": {},
        "Peers": [
            {
                "Name": "node0-f6a6f8e18a9d",
                "IP": "10.10.10.5"
            },
            {
                "Name": "node1-507a763bed93",
                "IP": "10.10.10.6"
            }
        ]
    }
]
```

Notice that the IP address listed for the service task (container) running on **node0** is different to the IP address for the service task running on **node1**. Note also that they are one the sane "overnet" network.

Run a `docker ps` command to get the ID of the service task on **node0** so that you can log in to it in the next step.

```
$ docker ps
CONTAINER ID        IMAGE                                                                            COMMAND                  CREATED             STATUS              PORTS                           NAMES
d676496d18f7        ubuntu@sha256:dd7808d8792c9841d0b460122f1acf0a2dd1f56404f8d1e56298048885e45535   "sleep infinity"         10 minutes ago      Up 10 minutes                                       myservice.2.nlozn82wsttv75cs9vs3ju7vs
<Snip>
```

Log on to the service task. Be sure to use the container `ID` from your environment as it will be different from the example shown below. We can do this by running `docker exec -it <CONTAINER ID> /bin/bash`.

```
$ docker exec -it d676496d18f7 /bin/bash
root@d676496d18f7:/#
```

Install the ping command and ping the service task running on **node1** where it had a IP address of `10.10.10.3` from the `docker network inspect overnet` command.

```
root@d676496d18f7:/# apt-get update && apt-get install -y iputils-ping
```

Now, lets ping `10.10.10.3`.

```
root@d676496d18f7:/# ping -c5 10.10.10.3
PING 10.10.10.3 (10.10.10.3) 56(84) bytes of data.
^C
--- 10.10.10.3 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 2998ms
```

The output above shows that both tasks from the **myservice** service are on the same overlay network spanning both nodes and that they can use this network to communicate.

## <a name="discover"></a>Step 5: Test service discovery

Now that you have a working service using an overlay network, let's test service discovery.

If you are not still inside of the container on **node0**, log back into it with the `docker exec -it <CONTAINER ID> /bin/bash` command.

Run `cat /etc/resolv.conf` form inside of the container on **node0**.

```
$ docker exec -it d676496d18f7 /bin/bash
root@d676496d18f7:/# cat /etc/resolv.conf
search ivaf2i2atqouppoxund0tvddsa.jx.internal.cloudapp.net
nameserver 127.0.0.11
options ndots:0
```

The value that we are interested in is the `nameserver 127.0.0.11`. This value sends all DNS queries from the container to an embedded DNS resolver running inside the container listening on 127.0.0.11:53. All Docker container run an embedded DNS server at this address.

> **NOTE:** Some of the other values in your file may be different to those shown in this guide.

Try and ping the "myservice" name from within the container by running `ping -c5 myservice`.

```
root@d676496d18f7:/# ping -c5 myservice
PING myservice (10.10.10.2) 56(84) bytes of data.
64 bytes from 10.10.10.2: icmp_seq=1 ttl=64 time=0.020 ms
64 bytes from 10.10.10.2: icmp_seq=2 ttl=64 time=0.052 ms
64 bytes from 10.10.10.2: icmp_seq=3 ttl=64 time=0.044 ms
64 bytes from 10.10.10.2: icmp_seq=4 ttl=64 time=0.042 ms
64 bytes from 10.10.10.2: icmp_seq=5 ttl=64 time=0.056 ms

--- myservice ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4001ms
rtt min/avg/max/mdev = 0.020/0.042/0.056/0.015 ms
```

The output clearly shows that the container can ping the `myservice` service by name. Notice that the IP address returned is `10.10.10.2`. In the next few steps we'll verify that this address is the virtual IP (VIP) assigned to the `myservice` service.

Type the `exit` command to leave the `exec` container session and return to the shell prompt of your **node0** Docker host.

```
root@d676496d18f7:/# exit
```

Inspect the configuration of the "myservice" service by running `docker service inspect myservice`. Lets verify that the VIP value matches the value returned by the previous `ping -c5 myservice` command.

```
$ docker service inspect myservice
[
    {
        "ID": "ov30itv6t2n7axy2goqbfqt5e",
        "Version": {
            "Index": 19
        },
        "CreatedAt": "2017-04-04T09:35:47.009730798Z",
        "UpdatedAt": "2017-04-04T09:35:47.05475096Z",
        "Spec": {
            "Name": "myservice",
            "TaskTemplate": {
                "ContainerSpec": {
                    "Image": "ubuntu:latest@sha256:dd7808d8792c9841d0b460122f1acf0a2dd1f56404f8d1e56298048885e45535",
                    "Args": [
                        "sleep",
                        "infinity"
                    ],
<Snip>
        "Endpoint": {
            "Spec": {
                "Mode": "vip"
            },
            "VirtualIPs": [
                {
                    "NetworkID": "wlqnvajmmzskn84bqbdi1ytuy",
                    "Addr": "10.10.10.2/24"
                }
            ]
        },
<Snip>
```

Towards the bottom of the output you will see the VIP of the service listed. The VIP in the output above is `10.10.10.2` but the value may be different in your setup. The important point to note is that the VIP listed here matches the value returned by the `ping -c5 myservice` command.

Feel free to create a new `docker exec` session to the service task (container) running on **node1** and perform the same `ping -c5 service` command. You will get a response form the same VIP.

## Wrap Up

Thank you for taking the time to complete this lab! Feel free to try any of the other labs.
