# Swarm  (single node, swarm manager node)

## 1. Create New Image

> [參考 - Get Started, Part 2: Containers] (https://docs.docker.com/get-started/part2/)

## 2. Create Service

~~~
$ pwd
/home/vagrant/swarm-demo
~~~

~~~
$ nano docker-compose.yml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: gecko719/friendlyhello:latest
    deploy:
      replicas: 6
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
~~~

啟動swarm manager

~~~
$ docker swarm init
~~~

> Note: We’ll get into the meaning of that command in part 4. If you don’t run docker swarm init you’ll get an error that “this node is not a swarm manager.”

透過docker-compose.yml啟動名為'hello'的service, 且產生6個replicas實例

~~~
$ docker stack deploy -c docker-compose.yml hello
~~~

檢視所有docker service服務名稱

~~~
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                           PORTS
x6h1y5u36bo6        hello_web           replicated          6/6                 gecko719/friendlyhello:latest   *:80->80/tcp
~~~

檢視docker service (指定service name)

~~~
$ docker service ps hello_web
ID                  NAME                IMAGE                           NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
usxb4g0bt6qb        hello_web.1         gecko719/friendlyhello:latest   localhost           Running             Running 24 minutes ago                       
vqjwzvfkpfvy        hello_web.2         gecko719/friendlyhello:latest   localhost           Running             Running 24 minutes ago                       
vs32crnrwt1b        hello_web.3         gecko719/friendlyhello:latest   localhost           Running             Running 24 minutes ago                       
xfttlk4ij8d1        hello_web.4         gecko719/friendlyhello:latest   localhost           Running             Running 24 minutes ago                       
l0wucoxw5yff        hello_web.5         gecko719/friendlyhello:latest   localhost           Running             Running 24 minutes ago                       
xrwrp1yoiwuh        hello_web.6         gecko719/friendlyhello:latest   localhost           Running             Running 24 minutes ago                       

~~~

檢視特定container資訊

~~~
$ docker inspect usxb4g0bt6qb
[
    {
        "ID": "usxb4g0bt6qblhl7b1in1kjzh",
    }
    TL;DR
]    
~~~

~~~
$ docker inspect --format='{{.Status.ContainerStatus.ContainerID}}' usxb4g0bt6qb
5db40a9a70f978908e771e0f383d08243d280fde06c9c7954ba2ee575cab46d3
~~~


Scale the app - 修改docker-compose.yml的replicas屬性值後，再次執行

~~~
$ docker stack deploy -c docker-compose.yml hello
~~~

Take down the app and the swarm

~~~
$ docker stack rm hello
~~~


