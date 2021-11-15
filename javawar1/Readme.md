### 包裝 Java War file with Tomcat to Docker Image
1. a simple war, just show HelloWorld string on page
![xxx](https://i.imgur.com/F5Ji7uD.png)
2. Dockerfile
```
From tomcat:jre8-alpine
LABEL maintainer="danc.yu@gmail.com"

ADD HelloWorldDocker.war /usr/local/tomcat/webapps/

EXPOSE 8080
CMD ["catalina.sh", "run"]


```
3. Create a Folder, named javawar1 and put the war file and Dockerfile
```buildoutcfg
javawar1
├── Dockerfile
└── HelloWorldDocker.war
```
4. 建立Docker image指令
```buildoutcfg
%cd javawar1
%docker build -t javawar1_image .
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
把image命名為 javawar1_image
5. 執行
```buildoutcfg
docker run -p 80:8080 javawar1_image
```
6. 確認服務
開啟瀏覽器 輸入 http://localhost/HelloWorldDocker/HelloWorld
![](https://i.imgur.com/fR4iCi8.png)