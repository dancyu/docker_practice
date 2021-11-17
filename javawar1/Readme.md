# 包裝 Java War file with Tomcat to Docker Image
###### tags: `Tag(docker)` `Tag(Tomcat)`

這是拿 java war檔由淺入深的實作Docker 練習筆記

### Agenda
包裝Docker/啟動Container的方式由簡入深:
- 陽春版
- 加入Volume
- 用docker volume 指定volume
- 使用docker-compose啟動container
- 指定container name
- docker-compose 啟動container放到背景執行
- container指定時區
---
### 陽春版
2. a simple war, just show HelloWorld string on page
![xxx](https://i.imgur.com/F5Ji7uD.png)
3. Dockerfile
```
From tomcat:jre8-alpine
LABEL maintainer="danc.yu@gmail.com"

ADD HelloWorldDocker.war /usr/local/tomcat/webapps/

EXPOSE 8080
CMD ["catalina.sh", "run"]

```
ps. LABEL maintainer is deprecated.

3. Create a Folder, named javawar1 and put the war file and Dockerfile
```buildoutcfg
javawar1
├── Dockerfile
└── HelloWorldDocker.war
```
4. 建立Docker image指令
```buildoutcfg
% cd javawar1
% docker build --no-cache-t -t javawar1_image .
[+] Building 3.8s (8/8) FINISHED
 => [internal] load build definition from Dockerfile                                                                         0.0s
 => => transferring dockerfile: 37B                                                                                          0.0s
 => [internal] load .dockerignore                                                                                            0.0s
 => => transferring context: 2B                                                                                              0.0s
 => [internal] load metadata for docker.io/library/tomcat:jre8-alpine                                                        3.6s
 => [auth] library/tomcat:pull token for registry-1.docker.io                                                                0.0s
 => [internal] load build context                                                                                            0.0s
 => => transferring context: 219B                                                                                            0.0s
 => [1/2] FROM docker.io/library/tomcat:jre8-alpine@sha256:04feaf74f8bb54b43ea136b150bbc7b58e8a3062aead67ab871f2dbbd5dac5d1  0.0s
 => CACHED [2/2] ADD HelloWorldDocker.war /usr/local/tomcat/webapps/                                                         0.0s
 => exporting to image                                                                                                       0.0s
 => => exporting layers                                                                                                      0.0s
 => => writing image sha256:fd9718cd200606846cf5854cdcb04d5cdb2c658a2a75e21c3fa411037c1125c0                                 0.0s
 => => naming to docker.io/library/javawar1_image
```
- 指定--no-cache -t 可避免cache
- -t image命名為 javawar1_image 指定版號 -t javawar1_image:v2

5. 啟動container
```buildoutcfg
docker run -p 80:8080 javawar1_image
```

6. 確認服務
開啟瀏覽器 輸入 http://localhost/HelloWorldDocker/HelloWorld
![](https://i.imgur.com/fR4iCi8.png)
---
### 加入Volume
- 上述啟動服務後，透過 Docker Dashboard 可直接點選該container CLI
![](https://i.imgur.com/gUkO5sP.png)
- 它會帶著docker exec -it {完整container_id} /bin/sh 進入container,就可以對container下linux指令。預設路徑就是tomcat工作路徑
![](https://i.imgur.com/Mn8Gw3R.png)
container內tomcat 在/usr/local/tomcat/logs 下產生log

- 緣由:
    - 若執行docker run,每次都起新的container，這些log就消失
    - 要到container內找log不方便，可以指定本機目錄取代container內的/usr/local/tomcat/logs。 (ps.查閱log應可結合其他監控軟體,後續再延伸。)
    - 應用程式除了Log之外，如資料庫還是需要有固定存取的磁碟空間 

- 作法：
  - 先在本機新增一個目錄 叫做volume/javawar1 
  - 在本機的路徑是 /Users/dancyu/codes/docker_practice_private/volume/javawar1
    - 這是啟動container時的方法要調整，不需重做image 故不用修改Dockerfile 
  - docker run執行
```buildoutcfg
docker run -p 80:8080 -v /Users/dancyu/codes/docker_practice_private/volume/javawar1:/usr/local/tomcat/logs javawar1_image
```
執行起來後，可以看到本機的目錄已有tomcat產生的檔案
![](https://i.imgur.com/mx6PTaD.png)
(這是新啟動的container所以不會有之前寫在container內的log)

---
### 用docker volume 指定volume
延續前文，透過docker volume 管理volume
- 指令格式
```docker volume create [OPTIONS] [VOLUME]```
- 指令
```buildoutcfg
% docker volume create javawar1_vol
javawar1_vol

觀察docker volume：
% docker volume inspect javawar1_log
[
    {
        "CreatedAt": "2021-11-16T03:17:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/javawar1_vol/_data",
        "Name": "javawar1_vol",
        "Options": {},
        "Scope": "local"
    }
]

啟動container
% docker run -p 80:8080 -v javawar1_vol:/usr/local/tomcat/logs javawar1_image


```
原目的是以docker volume 取代 上一小節的做法，但始終不成功[嘗試紀錄](https://hackmd.io/@CqRN13XlTXynDVuQ6clCkw/S1jOFKeuY)
目前練習結果只能由docker volume控制寫到本機位置，無法指定寫到哪裡，後續待深究。

- docker run執行
```buildoutcfg
docker run -p 80:8080 -v javawar1_ㄒㄠ:/usr/local/tomcat/logs javawar1_image
```
- 關於docker volume create 官網文件可參考此[連結](https://docs.docker.com/engine/reference/commandline/volume_create/)

---

### 改用docker compose啟動container
延續本專案，將docker run 改為 docker-compose的方法
- 在javawar1目錄下新增 docker-compose.yml 
```buildoutcfg
version: '3.7'
services:
  web:
    image: javawar1_image
    volumes:
      - javawar1_vol:/usr/local/tomcat/logs
    ports:
      - 80:8080

volumes:
  javawar1_vol:{javawar1_vol}
```
version 是指docker-compose yml格式版本，不是應用程式的版本，應用程式的版本可以定義在Dockerfile LABEL內。
image 指定使用產生的image,若不指定image 要改用build: . (docker_compose會參照同目錄下的
Dockerfile重新build image再啟動container)
- 執行
```buildoutcfg
docker-compose up
```
啟動後會發現
group 會以執行位置的當前目錄來命名，
![](https://i.imgur.com/qFxc3Nx.png)
關閉container後，再次執行docker-compose up也是同一個container

另外
原來預期改用docker-compose.yum指定volume,應該要使用同一塊磁碟，但並沒有，使用docker dashboard看javawar1_web_1的內容 發現Mount目錄名多了一個javawar1_javawar1_vol
![](https://i.imgur.com/3TFfGm2.png)
透過docker dashboard看Volumes, 發現 docker run 與docker-compose確實都使用不同volumes
![](https://i.imgur.com/mF6WoO7.png)
為了確認命名，我又多建了一個javawar2目錄,docker-compose.yml 把port改為81:8080,確定新增了一個javawar2_javawar1_vol

懷疑是否volume指定為local的特性只能讓一個container使用，驗證方式：用docker run
```
% docker run -p 82:8080 -v javawar1_vol:/usr/local/tomcat/logs javawar1_image

% docker run -p 83:8080 -v javawar1_vol:/usr/local/tomcat/logs javawar1_image
```
驗證結果: 多個container可以指向同一塊volume
![](https://i.imgur.com/vXG0Gz6.png)

結論：
1. volume 可以同時給多個container同時使用
    * 注意在這個練習拿tomcat當範例很不適合，正式環境不會把多個tomcat log導在一起, 本範例的實作目的是為了實驗container重啟時可以使用同一塊空間繼續寫入。
3. docker run 與docker-compose 啟動的container 指定volume時要注意目錄名 (但是否為我的docker-compose指定的不正確,未正確設定volumes map內容,待考證)
4. 用docker volume產生在/var/lib/docker/volumes/下的volume檔案只有在container啟動起來後才能使用cli去調閱內容

---
### 指定container name
在docker-compose.yml加上 container_name
```
version: '3.7'
services:
  web:
    container_name: javawar1_app
    image: javawar1_image
    volumes:
      - javawar1_vol:/usr/local/tomcat/logs
    ports:
      - 80:8080
volumes:
  javawar1_vol:
```
指定container name之後，如下
![](https://i.imgur.com/ZxpqBAD.png)

docker exec -it 也可指定container_name,不用去找container id
![](https://i.imgur.com/P4sHdyX.png)



### 啟動container放到背景執行
執行docker-cpmpose up 加上-d 背景執行，就不會每次都跑出log
```
docker-compose up -d
```

---
### container指定時區
tomcat的log內容都是utc時區, 非CST
在docker-compose.yml加上
```buildoutcfg
version: '3.7'
services:
  web:
    image: javawar1_image
    volumes:
      - javawar1_vol:/usr/local/tomcat/logs
    ports:
      - 80:8080
    environment:
      - TZ=Asia/Taipei
volumes:
  javawar1_vol:{javawar1_vol}
```
更改後，再用docker-compose up執行，發現log寫出的時間已是台北時區
![](https://i.imgur.com/98NrgBw.png)
但是時間還是UTC，google其他同步container與本機時間的方式，但沒有效果。
-v /etc/localtime:/etc/localtime:ro
```buildoutcfg
version: '3.7'
services:
  web:
    image: javawar1_image
    volumes:
      - javawar1_vol:/usr/local/tomcat/logs
      - /etc/localtime:/etc/localtime:ro
    ports:
      - 80:8080
    environment:
      - TZ=Asia/Taipei
volumes:
  javawar1_vol:{javawar1_vol}
```
思考：監控軟體抓取log的時間可以讀取utc並顯示為 cst, 或許可不去改container時間。目前還在嘗試單一docker的階段，設定TZ=Asia/Taipei已夠debug用。

### 待研究
1. docker run 或docker-compose up執行起container就直接看到tomcat log, 是否都應該改為背景執行？
    > docker-compose up -d 
    > 指定-d 就會是背景執行了 
3. 如何指定啟動都使用同一個container? 
    > 這問法有誤區，問題源自每次用docker run就產生一個新的container,但需要以使用情境來說明
    > 若以在本機使用docker db instead of install 並指定volume, 則可透過docker dashboard 起停 container
    > 用docker-compose up 若docker-compose.yml有更新,docker會destroy舊的container,再起一個新的.看起來像是原來的那一個 但container_id已是不同. 指定volume則會讓寫出磁碟有延續性。 
4. docker-compose.yml volumes 的設定方式待確認 {}的設定方法
5. war file升版的處理流程
6. tomcat upgrade
7. application container 應該是stateless的使用,隨時可以重啟AAA橫向擴充.  需要思考application的異常監控機制
8. Dockerfile內寫了cmd, docker-compose.yml也可以下EXEC 這兩個在實務上要怎麼搭配使用，需要再深入了解兩者間的差異。

