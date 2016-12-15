---
layout: post
title: JAVA项目docker发布
date: 2015-11-15
tags: DOCKER   
---

docker 发布JAVA项目

* 1、build.gradle 文件中增加如下

```
task buildDocker(type: Docker, dependsOn: build) {
    push = false
    applicationName = jar.baseName
    dockerfile = file('src/main/docker/Dockerfile')

    doFirst {
        copy {
            from (jar) {
                include "*${version}.jar"
                rename "(.*)-${version}(.*)", 'app.jar'
            }
            into stageDir
        }
    }
    dryRun = true
}
afterReleaseBuild.dependsOn buildDocker
```

* 2、编辑 src/main/docker/Dockerfile

```
FROM java:8-jre-alpine
VOLUME /tmp

ENV APP_HOME /usr/local/app

RUN mkdir -p "$APP_HOME"
WORKDIR $APP_HOME

ADD app.jar app.jar
RUN sh -c "touch app.jar"
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","app.jar"]
EXPOSE 8080
```
