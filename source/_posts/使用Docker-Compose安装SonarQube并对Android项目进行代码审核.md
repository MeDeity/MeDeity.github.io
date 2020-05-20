---
title: 使用SonarQube对Android项目进行代码审查
date: 2020-05-09 15:05:16
categories: 工具使用
tags: 代码审查 
---
#### sonarqube服务器安装
采用的是docker-compose进行安装,以下脚本来源于
[sonarqube-docker-compose](https://github.com/cookcodeblog/OneDayDevOps/blob/master/components/sonarqube/docker-compose.yml)
```
version: "2"

services:
  sonarqube:
    image: sonarqube:6.7.1
    restart: always
    ports:
      - "9000:9000"
    depends_on:
      - db
    networks:
      - sonarnet
    environment:
      - sonar.jdbc.username=sonar
      - sonar.jdbc.password=sonar123
      - sonar.jdbc.url=jdbc:postgresql://db:5432/sonarqube
      - SONARQUBE_JDBC_USERNAME=sonar
      - SONARQUBE_JDBC_PASSWORD=sonar123
      - SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonarqube
    volumes:
      - /srv/docker/sonarqube/sonarqube_conf:/opt/sonarqube/conf
      - /srv/docker/sonarqube/sonarqube_data:/opt/sonarqube/data
      - /srv/docker/sonarqube/sonarqube_extensions:/opt/sonarqube/extensions

  db:
    image: postgres:9.6
    restart: always
    networks:
      - sonarnet
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar123
      - POSTGRES_DB=sonarqube
    volumes:
      - /srv/docker/sonarqube/postgresql:/var/lib/postgresql
      - /srv/docker/sonarqube/postgresql_data:/var/lib/postgresql/data

networks:
  sonarnet:
    driver: bridge
```
>请确保安装了docker及docker-compose,安装完成后,执行以下命令进行安装
```
COMPOSE_HTTP_TIMEOUT=120 docker-compose up
```
运气好的话,sonarqube服务器安装完成,可通过 http://ipAddress:9000 访问.



#### 在Gradle(Android项目)中集成sonarqube
#### 1.在Project级的Build.gradle中添加以下内容(共添加#6处)
```gradle
buildscript {
    
    repositories {
        ...//其他项目用到的repositories
        maven { url "https://plugins.gradle.org/m2/"} //#1
    }
    dependencies {
        ...//其他项目用到的dependencies
        classpath ("org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:2.6")//#2

    }
}

//紧挨buildscript下方(位置有要求?) #3
plugins {
    id "org.sonarqube" version "2.6"
}

apply plugin: "org.sonarqube" //#3

//#5
sonarqube {
    properties {
        property "sonar.sourceEncoding", "UTF-8"
    }
}

//#6
subprojects {
    sonarqube {
        properties {
            property "sonar.sources", "src/main/java"
        }
    }
}

```

#### 认证权限信息
在gradle.properties文件中添加sonarqube服务器地址及登录token
```
# gradle.properties
systemProp.sonar.host.url=http://ipAddress:9000

#----- Token generated from an account with 'publish analysis' permission
systemProp.sonar.login= xxxxxxxxxxxxxxxxxxxxxxxxxxx
```

#### 运行命令(window平台)
```
gradlew sonarqube
```

#### 安装期间遇到了什么问题
Q:`执行docker-compose up -d时出现ERROR: Failed to Setup IP tables: Unable to enable SKIP DNAT rule`
##### 如何解决
通过重启docker 服务解决
```
service docker restart
```


Q:`Error creating container: UnixHTTPConnectionPool(host='localhost', port=None): Read timed out. (read timeout=60)`

##### 如何解决
通过以下命令解决了超时的问题
```shell
COMPOSE_HTTP_TIMEOUT=120 docker-compose up
```
[issuecomment-408199226](https://github.com/docker/compose/issues/4459#issuecomment-408199226)

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
