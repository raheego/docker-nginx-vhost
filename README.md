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

### step 4

- lb 폴더에도 default.conf 복사하기 
```bash
 $sudo docker cp config/default.conf lb:/etc/nginx/conf.d
```

### step 5
-  mv config lb 로 옮기기 
```
.
├── README.md
├── lb
│   └── config
│       └── default.conf
├── serv-a
│   └── index.html
└── serv-b
    └── index.html

```
### step 6
```
$ vi serv-a/index.html  <h1>A</h1>
$ cp serv-a/index.html serv-b
$ vi serv-b/index.html  <h1>B</h1>
각각 파일 만들어서 컨테이너 /usr/share/nginx/html/ 밑에 각각 cp하기
$ sudo docker cp serv-a/index.html serv-a:/usr/share/nginx/html/
$ sudo docker cp serv-b/index.html serv-b:/usr/share/nginx/html/
```
```
 $ sudo docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                   NAMES
00973b69b91e   nginx     "/docker-entrypoint.…"   52 minutes ago   Up 52 minutes   0.0.0.0:8003->80/tcp, :::8003->80/tcp   serv-b
2282216488a5   nginx     "/docker-entrypoint.…"   54 minutes ago   Up 54 minutes   0.0.0.0:8002->80/tcp, :::8002->80/tcp   serv-a
```


## ref
- https://hub.docker.com/_/nginx
