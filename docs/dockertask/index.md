# Docker Task
## Description
Create 3 containers named pc1, pc2, and pc3. The container pc1 should act as a router, allowing pc2 and pc3 to communicate with each other, as well as access the internet.

## Install docker
#### using the apt repository
1. Set up Docker's apt repository.
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
2. Install the Docker packages.
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Task
#### Create 3 network
```
docker network create  br0 --subnet 192.168.44.0/24
docker network create  ht0 --subnet 192.168.66.0/24
docker network create  ht1 --subnet 192.168.77.0/24
```
#### Tree of directories
```
├── docker-compose.yml
├── pc1
│   ├── Dockerfile
│   └── script.sh
├── pc2
│   ├── Dockerfile
│   └── script.sh
└── pc3
    ├── Dockerfile
    └── script.sh
```
#### docker-compose.yml
You can use docker-compose in 2 way:
!!! NOTE
    Option 2 is recommended

1. Create images from Dockerfiles then run
```
services:
  pc1:
    container_name: pc1
    build: ./pc1
    cap_add:
      - NET_ADMIN
    networks:
      br0:
        ipv4_address: 192.168.44.101
      ht0:
        ipv4_address: 192.168.66.101
      ht1:
        ipv4_address: 192.168.77.101
    tty: true
  pc2:
    container_name: pc2
    build: ./pc2
    cap_add:
      - NET_ADMIN
    networks:
      ht0:
        ipv4_address: 192.168.66.102
    tty: true
  pc3:
    container_name: pc3
    build: ./pc3
    cap_add:
      - NET_ADMIN
    networks:
      ht1:
        ipv4_address: 192.168.77.102
    tty: true
networks:
  br0:
    external: true
  ht0:
    external: true
  ht1:
    external: true
```

2. Use docker-compose.yml with created images
```
services:
  pc1:
    container_name: pc1
    image: javadsafari81/pc1:latest
    cap_add:
      - NET_ADMIN
    networks:
      br0:
        ipv4_address: 192.168.44.101
      ht0:
        ipv4_address: 192.168.66.101
      ht1:
        ipv4_address: 192.168.77.101
    tty: true
  pc2:
    container_name: pc2
    image: javadsafari81/pc2:latest
    cap_add:
      - NET_ADMIN
    networks:
      ht0:
        ipv4_address: 192.168.66.102
    tty: true
  pc3:
    container_name: pc3
    image: javadsafari81/pc3:latest
    cap_add:
      - NET_ADMIN
    networks:
      ht1:
        ipv4_address: 192.168.77.102
    tty: true
networks:
  br0:
    external: true
  ht0:
    external: true
  ht1:
    external: true
```

#### PC1 directory: 

Dockerfile:
```
FROM alpine:3.20.3

USER root:root

COPY script.sh /script.sh

RUN chmod +x /script.sh

ENTRYPOINT ["/script.sh"]
```

script.sh:
```
#!/bin/sh
ip route del 0.0.0.0/0
ip route add 0.0.0.0/0 via 192.168.44.100
apk update
apk add iptables
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
exec bin/sh
exec "$@"
```

#### PC2 directory:

Dockerfile:
```
FROM alpine:3.20.3

USER root:root

COPY script.sh /script.sh

RUN chmod +x /script.sh

ENTRYPOINT ["/script.sh"]
```

script.sh:
```
#!/bin/sh
ip route del 0.0.0.0/0
ip route add 0.0.0.0/0 via 192.168.66.101
exec bin/sh
exec "$@"
```

#### PC3 directory:

Dockerfile:
```
FROM alpine:3.20.3

USER root:root

COPY script.sh /script.sh

RUN chmod +x /script.sh

ENTRYPOINT ["/script.sh"]
```

script.sh:
```
#!/bin/sh
ip route del 0.0.0.0/0
ip route add 0.0.0.0/0 via 192.168.77.101
exec bin/sh
exec "$@"
```

#### Connecting to shell of container
Replace x with target number of pc
```
docker exec -it pcx sh
```
!!! Note
    I use script.sh because it wasn't possible for me to run commands that change network configuration during the image build stage due to permission errors. So, I build the image first, then run the script.

## Git
You can find files in my github --> dockerfile submodule
```
https://github.com/JavadSafari81/taskdoc
```
