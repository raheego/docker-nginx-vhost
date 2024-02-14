## docekr nginx vhost

## docker Load Balancing
![image](https://github.com/raheego/docker-nginx-vhost/assets/54056684/a362f2bb-e476-48b3-8e64-46f63245d80a)
- https://www.nginx.com/resources/glossary/load-balancing/

### step 1
- docker rm * rmi
```
$ sudo docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
$ sudo docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### step2
```bash
$ docker run -itd -p 8002:80 --name serv-a nginx
$ docker run -itd -p 8003:80 --name serv-b nginx
$ docker run -itd -p 8001:80 --name lb nginx:latest

$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                   NAMES
2fe6b3ebdb6e   nginx          "/docker-entrypoint.…"   5 seconds ago    Up 4 seconds    0.0.0.0:8002->80/tcp, :::8002->80/tcp   serv-a
fafba17b8cf6   nginx          "/docker-entrypoint.…"   18 seconds ago   Up 17 seconds   0.0.0.0:8003->80/tcp, :::8003->80/tcp   serv-b
fd3ca54a154a   nginx:latest   "/docker-entrypoint.…"   3 minutes ago    Up 3 minutes    0.0.0.0:8001->80/tcp, :::8001->80/tcp   lb
```

### step 3
- /home/rahee/code/docker-nginx-vhost 위치에 config 폴더 생성 후 default.conf 파일 생성
  
```
upstream serv {
        server serv-a:80;
        server serv-b:80;
}
server {
        listen 80;

        location /
        {
                proxy_pass http://serv;
        }
}
```

### setp 4

- lb 폴더에도 default.conf 복사하기 
```bash
 $sudo docker cp config/default.conf lb:/etc/nginx/conf.d
```

## ref
- https://hub.docker.com/_/nginx
