## docker compose


## docker engine
The docker has 3 components (layers)
- docker CLI  
    the cmd line interface. Doesn't need to be on the same host with other components. Use ```docker -H=10.123.2.1:2375 run nginx``` to specify the host.
- REST API server: API for programs to talk to the daemon
- docker daemon: a background process that manages docker objects (images, containers, ...)

### How does docker work under the hood?
Use namespaces to isolate workspace. Process ID, IPC, network, mount, etc. are created in their own namespace.

### Process ID Namespace
In the host's Linux system, there's a root process whose PID is 1.
In the container, there's also a logically root process whose PID is 1.
But processes running inside the container are in fact processes running on the underlying host. So processes with container PID 1 may have system PID 5. 

### cgroup
By default, there's no limitation on how much resource a container can use.
Use cgroups (control groups) to restrict the resource allocated to each container.
i.e., use ```docker run --cpu=.5 ubuntu``` to ensure the container won't use over 50% of the cpu, use ```docker run --memory=100m ubuntu``` to ensure it won't use more that 100 MB of memory.

## Docker Storage

### Layered architecture
Docker builds images in a layered architecture. Each instruction in the Dockerfile created a new layer with just the changes from the previous layer.
i.e.,
1. Base Ubuntu Layer         - 120 MB
2. Changes in apt packages   - 306 MB
3. Changes in pip packages   - 6.3 MB
4. Source code               - 229 B
5. Update Entry point        - 0 B

If there's another Dockerfile that has the same first three instructions, Docker won't build the first three layers. 

When a container is run, Docker creates a 6th layer to store data used by the container such as log files. The life of this layer is only as long as the container's life.

The image layers are read-only. If one wants to modify the source code in the image layer, Docker would automatically create a copy of the source code in the container layer, which would be edited by that user.

### volume mount
Use this command to create a volume at /var/lib/docker/volumes/data_volume:
```
docker volumne create data_volume
```

mount that volume by:
```
docker run -v data_volume:/var/lib/mysql mysql
```
/var/lib/mysql is the default position for mysql in the container

### bind mount
or, bind mount an existing directory to the container
```
docker run -v /data/mysql:/var/lib/mysql mysql
```

-v is old, use --mount as a preferred way to mount
```
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```
Docker uses storage drivers to enable layered architecture, depending on the underlying system.

## networking
### default networks
3 networks created when installing Docker: 
- brige: the default network a container attach to
- none
- host

use this command to specify the network
```docker run --network host Uduntu```

The bridge is an internal network on the host. The containers' IP usually starts from 172.17...

2 ways to access the containers from outside:
- Map host port to container port
- Associate the container to the host network. There will be no network isolation between the container and the host. Therefore multiple containers cannot be run on the same host on the same port.

### user-defined networks
create new internal network:
```docker network create --driver bridge --subnet 182.18.0.0/16 --gateway 182.18.0.1 custom-isolated-network```

list all networks:
```docker network ls```

see network settings and the IP address of a container:
```docker inspect <container_name/container_id>```

inspect a network:
```docker network inspect <network_name/network_id>```

### embedded DNS
Containers can reach each other using their names. Docker has a built-in DNS server.  
It's not gauranteed that the containers will get the same IP when the system reboots.  
Docker use network namespaces to create a namespace for each container. Then it users virtual Ethernet pairs to connect containers together.

## Docker Registry
Take ```docker run nginx``` as an example. In fact `nginx` contains three parts: `docker.io/nginx/nginx` (`registry/user account/image repository`).  
If using a private registry, first login to the registry
```
docker login private-registry.io
```
then 
```
docker run private-registry.io/apps/internal-app
```
The user can even deploy a private registry, as it's just another docker image:
```
docker run -d -p 5000:5000 --name registry registry:2
```
then
```
docker image tag my-image localhost:5000/my-image

docker push localhost:5000/my-image

docker pull localhost:5000/my-image
dokcer pull 192.168.56.100:5000/my-image
```

