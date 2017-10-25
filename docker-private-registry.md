# Docker Registry


```
撰寫日期：2017/09/14 - 
修改日期：
```

## 參考文章

1. [Docker Registry](https://docs.docker.com/registry/)
2. [Docker Registry RESTful API Spec](https://docs.docker.com/registry/spec/api/#listing-repositories)


## 1.建立Registry Container服務

透過docker-compose檔案建立Registry服務

~~~
$ nano docker-compose-registry.yaml 
registry:
  restart: always
  image: registry:2
  container_name: registry-dev
  ports:
    - 5000:5000
  volumes:
    - /docker/registry/data:/var/lib/registry
  environment:
    REGISTRY_STORAGE_DELETE_ENABLED: 'true'
~~~    

> PS: 設定 REGISTRY_STORAGE_DELETE_ENABLED 為 'true'，得以透過API刪除registry內image檔

## 2.建立Registry Container

建立Registry

~~~
$ docker-compose -f docker-compose-registry.yaml up -d
~~~

查看Registry

~~~
$ docker ps | grep registry
adaa0e32fc8d        registry:2                              "/entrypoint.sh /e..."   5 weeks ago         Up 8 days           0.0.0.0:5000->5000/tcp                                                    registry-dev
~~~


## 3.建立image
> 以busybox為例 （docker hub）

pull image - busybox

~~~
$ docker pull busybox
~~~

~~~
$ docker images | grep busybox
busybox                          latest                   efe10ee6727f        3 months ago        1.13 MB
~~~

重新建立標籤為 '127.0.0.1:5000/busybox:1.0'

~~~
$ docker tag busybox 127.0.0.1:5000/busybox:1.0
~~~

~~~
$ docker images | grep busybox
127.0.0.1:5000/busybox           1.0                      efe10ee6727f        3 months ago        1.13 MB
busybox                          latest                   efe10ee6727f        3 months ago        1.13 MB
~~~

## 4.上傳image至Registry

~~~
$ docker push 127.0.0.1:5000/busybox:1.0
The push refers to a repository [127.0.0.1:5000/busybox]
08c2295a7fa5: Pushed 
1.0: digest: sha256:2605a2c4875ce5eb27a9f7403263190cd1af31e48a2044d400320548356251c4 size: 527
~~~

## 5.查詢Registry內所有images名稱

~~~
$ curl http://127.0.0.1:5000/v2/_catalog
{"repositories":["busybox"]}
~~~

## 6.查詢特定image下所有tags

~~~
$ curl http://127.0.0.1:5000/v2/busybox/tags/list
{"name":"busybox","tags":["1.0"]}
~~~

## 7.查詢特定image+指定tag之manifests

~~~
$ curl --head -H "Accept: application/vnd.docker.distribution.manifest.v2+json" http://127.0.0.1:5000/v2/busybox/manifests/1.0
HTTP/1.1 200 OK
Content-Length: 527
Content-Type: application/vnd.docker.distribution.manifest.v2+json
Docker-Content-Digest: sha256:2605a2c4875ce5eb27a9f7403263190cd1af31e48a2044d400320548356251c4
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:2605a2c4875ce5eb27a9f7403263190cd1af31e48a2044d400320548356251c4"
X-Content-Type-Options: nosniff
Date: Wed, 25 Oct 2017 01:12:36 GMT

~~~

## 8.刪除特定image(需指定tag）

> 請複製第7步驛之Docker-Content-Digest值 'sha256:2605a2c4875ce5eb27a9f7403263190cd1af31e48a2044d400320548356251c4'

~~~
$ curl -X DELETE http://127.0.0.1:5000/v2/busybox/manifests/sha256:2605a2c4875ce5eb27a9f7403263190cd1af31e48a2044d400320548356251c4
~~~

### 確認是否刪除成功

catalog 'busybox' 仍存在

~~~
$ curl http://127.0.0.1:5000/v2/_catalog
{"repositories":["busybox"]}
~~~

但版本1.0己被刪除

~~~
$ curl http://127.0.0.1:5000/v2/busybox/tags/list
{"name":"busybox","tags":null}
~~~



## 9.Registry Container 操作
### 停止Registry Container

~~~
$ docker stop registry-dev
~~~

### 啟動Registry Container

~~~
$ docker start registry-dev
~~~

### 刪除Registry Container

~~~
$ docker-compose -f docker-compose-registry.yaml down
~~~


