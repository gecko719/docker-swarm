# Swarm Mode

### 主機配置 

|   IP    | Swarm Manager|  Worker |
|:-------:|:------------:|:-------:|
|10.0.1.17|   Y          |         |
|10.0.1.18|              |     Y   |


## 1. 配置Docker Swarm

### 1.1 初始Swarm Manager

在10.0.1.17主機上執行

~~~
vagrant@manager1 ~ $ docker swarm init --advertise-addr 10.0.1.17
Swarm initialized: current node (8i9vejajn7gq56xy63ganpstx) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4shp3kp283594e721ux78ti266odx2abedqazqo90g1s7x1k60-0ldn5zbxzveeevh315jrnv7jj 10.0.1.17:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
~~~

若想查詢加入其它主機成為swarm manager的指令，可執行

~~~
vagrant@ manager1 ~ $ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4shp3kp283594e721ux78ti266odx2abedqazqo90g1s7x1k60-5253qw7wmtmis1u88q8gvpd5q 10.0.1.17:2377
~~~    

若想查詢加入其它主機成為worker的指令，可執行 

~~~
vagrant@manager1 ~ $ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4shp3kp283594e721ux78ti266odx2abedqazqo90g1s7x1k60-0ldn5zbxzveeevh315jrnv7jj 10.0.1.17:2377
~~~


### 1.2 配置Swarm Worker Node

> 在10.0.1.18主機上執行下列指令

~~~
vagrant@worker1 ~ $ docker swarm join --token SWMTKN-1-4shp3kp283594e721ux78ti266odx2abedqazqo90g1s7x1k60-0ldn5zbxzveeevh315jrnv7jj 10.0.1.17:2377

This node joined a swarm as a worker.
~~~


### 1.3 查詢目前所有節點狀況

請於Swarm Manager下執行指令

~~~
vagrant@manager1 ~ $ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
8i9vejajn7gq56xy63ganpstx *   manager1            Ready               Active              Leader
y7j2vfsnk9n03itzlwaw9lffz     worker1             Ready               Active              
~~~

## 2. Deploy Service

### 2.1 Deploy s service to the swarm

~~~
vagrant@manager1 ~ $ docker service create --replicas 1 --name helloworld alpine ping docker.com
lfihnrievxb8aok3rgcvkvi9b
~~~

### 2.2 查詢deploy結果

~~~
vagrant@manager1 ~ $ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
lfihnrievxb8        helloworld          replicated          1/1                 alpine:latest     
~~~


~~~
vagrant@manager1 ~ $ docker service inspect --pretty lfihnrievxb8
或
vagrant@manager1 ~ $ docker service inspect --pretty helloworld

ID:		lfihnrievxb8aok3rgcvkvi9b
Name:		helloworld
Service Mode:	Replicated
 Replicas:	1
Placement:
UpdateConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:		alpine:latest@sha256:f006ecbb824d87947d0b51ab8488634bf69fe4094959d935c0c103f4820a417d
 Args:		ping docker.com 
Resources:
Endpoint Mode:	vip

~~~

~~~
vagrant@manager1 ~ $ docker service ps lfihnrievxb8
或
vagrant@manager1 ~ $ docker service ps helloworld

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
ykoczpaexoei        helloworld.1        alpine:latest       manager1            Running             Running 2 minutes ago                       
~~~

由上述指令得知helloworld Service是佈署在manager1 節點上，所以可以在10.0.1.17上執行下列指令

> 若在10.0.1.18上會查詢不到

~~~
vagrant@manager1 ~ $ docker ps | grep helloworld
8955130910f8        alpine:latest            "ping docker.com"        11 minutes ago      Up 11 minutes                                helloworld.1.ykoczpaexoei475utqe7tqnql
~~~

### 2.3 Scale Out

~~~
vagrant@manager1 ~ $ docker service scale helloworld=3
helloworld scaled to 3
~~~

查詢結果，worker1節點上多了2台container

~~~
vagrant@manager1 ~ $ docker service ps helloworld
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
ykoczpaexoei        helloworld.1        alpine:latest       manager1            Running             Running 14 minutes ago                       
15bc9gz66r7y        helloworld.2        alpine:latest       worker1             Running             Running 26 seconds ago                       
ss41zeb36xsi        helloworld.3        alpine:latest       worker1             Running             Running 26 seconds ago                       
~~~

所以可以在10.0.1.18上可查詢到2個helloworld containers

~~~
vagrant@worker1 ~ $ docker ps | grep helloworld
fc8d10c6b890        alpine:latest            "ping docker.com"        2 minutes ago       Up 2 minutes                                 helloworld.2.15bc9gz66r7yx4og4oipprgo9
c1f095031513        alpine:latest            "ping docker.com"        2 minutes ago       Up 2 minutes                                 helloworld.3.ss41zeb36xsi8aibwijlgglo1
~~~

## 3. Delete the service

### 3.1 Delete the service running on the swarm 

~~~
vagrant@manager1 ~ $ docker service rm helloworld
helloworld
~~~

