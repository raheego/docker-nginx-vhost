## docekr nginx vhost

### try dev branch ~


## docker Load Balancing
![image](https://github.com/raheego/docker-nginx-vhost/assets/54056684/a362f2bb-e476-48b3-8e64-46f63245d80a)
![image](https://github.com/raheego/docker-nginx-vhost/assets/54056684/610b5ecf-b1d7-4fa0-bbc1-a9a78f734d72)

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
- bridge 네트워크는 하나의 호스트 컴퓨터 내에서 여러 컨테이너 연결
- host 네트워크는 컨터이너를 호스트 컴퓨터와 동일한 네트워크에서 컨테이너를 돌리기 위해서 사용
- overlay 네트워크는 여러 호스트에 분산되어 돌아가는 컨테이너들 간에 네트워킹을 위해서 사용

```
$ docker network ls


 $  sudo docker network ls
 $  sudo docker network create abc
 
$ sudo docker network inspect abc
[
    {
        "Name": "abc",
        "Id": "890893d530e45b36c7c7c01e465773935b1bf5e2fd79e3a63b683c966f8a5609",
        "Created": "2024-02-14T12:48:07.928438906+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]


 $ sudo docker network inspect host
[
    {
        "Name": "host",
        "Id": "a1f6e3af6ac79d9cf675f18ec804e1932d6de94a89edd033489d3900a9c88f10",
        "Created": "2024-01-29T14:27:58.523572411+09:00",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": null
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```
```
connect
$  sudo docker network connect abc serv-a
$  sudo docker network connect abc serv-b
$  sudo docker network connect abc lb

$  sudo docker network inspect abc

[
    {
        "Name": "abc",
        "Id": "890893d530e45b36c7c7c01e465773935b1bf5e2fd79e3a63b683c966f8a5609",
        "Created": "2024-02-14T12:48:07.928438906+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "00973b69b91e1bf2dadd610db75f366344e4476e87b906d8256e0c0b5afb37bc": {
                "Name": "serv-b",
                "EndpointID": "8fa424cceec9fb39b988c06a1fd2a9235b5a756a326125a78b110524b87a2de7",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "03ea8ce41073136024943447964f41c03a2196bdff99142329e05f327bdd2239": {
                "Name": "lb",
                "EndpointID": "46a2860c4d1fc104a65a6cd9705c4a0c7da322091848ec8fe9a7cfc36dfe78b7",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "2282216488a515eaefd3f4867df46b9fab7b4929d22f85b20fd76958c6176f77": {
                "Name": "serv-a",
                "EndpointID": "e487e42e02ae059bc69cb80ef68beaf808f8e87d608594c9ff2373b582ca1eda",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

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


### Try 2(Dockerfile 생성 후 build, run)
```
- 도커 파일 생성 
$ vi lb/Dockerfile
From nginx
COPY config/default.conf /etc/nginx/conf.d/

$ vi serv-a/Dockerfile 
FROM nginx
COPY index.html /usr/share/nginx/html

$ vi serv-b/Dockerfile 
FROM nginx
COPY index.html /usr/share/nginx/html

build
$ sudo docker build -t serv-b:\n0.1.0 .
$ sudo docker build -t serv-b:0.1.0 .
$ sudo docker build -t lb:0.1.0 .

run
$ sudo docker run -d --name lb -p 8001:80 lb:0.1.0
$ sudo docker run -d --name serv-a serv-a:0.1.0
$ sudo docker run -d --name serv-b serv-b:0.1.0

network
$ docker network create dockerfileNW
$ sudo docker network connect dockerfileNW serv-a
$ sudo docker network connect dockerfileNW serv-b
$ sudo docker network connect dockerfileNW lb
```

## ref
- https://hub.docker.com/_/nginx
- https://github.com/pySatellite/docker-nginx-vhost
