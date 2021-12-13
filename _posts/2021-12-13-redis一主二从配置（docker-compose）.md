
先创建redis集群的网络： 
```
  docker network create redis-net
```

编写`docker-compose.yml`文件，如下： 

```
version: "3"

# 定义服务，可以多个
services:
  redis-6371: # 服务名称
    image: redis # 创建容器时所需的镜像
    container_name: redis-6371 # 容器名称
    restart: always # 容器总是重新启动
    volumes: # 数据卷，目录挂载
      - $PWD/6371/data:/data
    command: redis-server --requirepass 123456
    networks:
      - redis-net

  redis-6372:
    image: redis
    container_name: redis-6372
    volumes:
      - $PWD/6372/data:/data
    command: redis-server --slaveof redis-6371 6379 --requirepass 123456 --masterauth 123456 
    depends_on:
      - redis-6371
    networks:
      - redis-net

  redis-6373:
    image: redis
    container_name: redis-6373
    volumes:
      - $PWD/6373/data:/data
    command: redis-server --slaveof redis-6371 6379 --requirepass 123456 --masterauth 123456
    depends_on:
      - redis-6371
    networks:
      - redis-net

networks:
  redis-net:
    external: true
```
