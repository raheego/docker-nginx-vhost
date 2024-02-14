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
- lb 에 각각 담겨 있기 때문에 a,b 번갈아가면서 화면 노출됨
- lb가 대리자 프록시 역할

### step 7
- https://github.com/raheego/docker-nginx-vhost/issues/2

  

## Try
- serv-a 와 serv-b의 포트번호인 8002,8003 을 없애고, 8001 포트인 lb만을 통해서 접속하는 실습
- pull 하지 않고 commit만 해도 됨

1. serv-a, serv-b commit 해서 image로 만들기
2. serv-a, serv-b 삭제하기
3. serv-a serv-b run 하기 (-p옵션 X)
4. serv-a serv-b 다시 abc로 connect해주기

```
$ sudo docker commit serv-a rahee/serv-a
$ sudo docker commit serv-b rahee/serv-b

$ docker rm serv-a
$ docker rm serv-b

$ sudo docker run --name serv-a -d rahee/serv-a  // -p옵션 없이 run 하기
$ sudo docker run --name serv-b -d rahee/serv-b  // -p옵션 없이 run 하기

$ sudo docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS             PORTS                                   NAMES
11b057d6e7ec   rahee/serv-b   "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes       80/tcp                                  serv-b
a18cd1a18fff   rahee/serv-a   "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes       80/tcp                                  serv-a
03ea8ce41073   nginx:latest   "/docker-entrypoint.…"   4 hours ago     Up About an hour   0.0.0.0:8001->80/tcp, :::8001->80/tcp   lb

$ sudo docker network connect abc serv-a
$ sudo docker network connect abc serv-b

$ sudo docker network inspect abc // 컨테이너 확인 
```


## ref
- https://hub.docker.com/_/nginx
