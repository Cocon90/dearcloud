### Docker搭建镜像仓库和配置缓冲地点

    参考网址：https://docs.docker.com/engine/reference/commandline/dockerd/#options

#### 一、配置Docker镜像仓库

1、新建docker-compose.yml，内容如下：
```yml
version: '3'
services:
  docker_registry:
    image: registry:2.7
    ports:
      - 0.0.0.0:22000:5000
    volumes:
      - /mnt/disk/docker/registry:/var/lib/registry
    deploy:
      placement:
        constraints:
          - node.hostname==host-10-126-141-22
      restart_policy:
        condition: on-failure
```
2、创建DockerSwarm集群
``` bash
    docker swarm init --advertise-addr 10.126.141.22
```

3、部署镜像仓库
```bash
    docker stack deploy -c docker-compose.yml docker-registery
```

4、查看运行情况
``` bash
    docker stack ps docker-registery
```

5、停止服务
``` bash
    docker stack rm docker-registery
```


#### 二、配置Docker镜像存储地点，和配置局域网内的不安全镜像仓库：

1、创建配置
```bash
sudo mkdir /mnt/disk/docker/dataroot
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "insecure-registries": ["10.126.141.22:22000"],
  "data-root":"/mnt/disk/docker/dataroot"
}
EOF
```

2、重新加载配置
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

3、查看是否已修改
```bash
sudo docker info | grep Root
```

