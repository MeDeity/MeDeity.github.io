---
title: '[uni-app]iOS云打包如何设置通用链接'
date: 2020-03-31
categories: 
  - uni-app
tags:
  - 技巧
---
##### 通用链接（Universal Link）是什么

Universal Link是苹果在WWDC 2015上提出的iOS 9的新特性之一。此特性类似于深层链接，并能够方便地通过打开一个Https链接来直接启动您的客户端应用(手机有安装App)。对比起以往所使用的URL Sheme, 这种新特性在实现web-app的无缝链接时能够提供极佳的用户体验。那这么神(繁)奇(琐)的功能在UNI-APP中如何配置实现呢

##### 二、开启Associated Domains服务

在"Certificates, Identifiers & Profiles"页面选择"Identifiers"中选择对应的App ID，确保开启Associated Domains服务,开启Associated Domains服务后需要重新生成profile文件，提交云端打包时使用

##### 三、使用HBuilderX云端打包时在manifest.json中配置域名

uni-app项目在"app-plus" -> "distribute" -> "ios" -> "capabilities" -> "entitlements"[如果没有则自行添加]下添加"com.apple.developer.associated-domains"字段，字段值为字符串数组，每个字符串为要关联的域名

```json
"capabilities": {  
    "entitlements": {  
        "com.apple.developer.associated-domains": [  
            //待绑定的域名 demo.dcloud.net.cn
            "applinks:demo.dcloud.net.cn"  
        ]  
    }  
}
```

一、服务器配置apple-app-site-association文件
在域名对应的服务器下放置apple-app-site-association文件

```
{  
  "applinks": {  
      "apps": [],  
      "details": [  
          {  
              "appID": "G56NU654TV.io.dcloud.HBuilder",  
              "paths": [ "/ulink/*"]  
          }  
      ]  
  }  
}
```

参数说明:

1. apps: 必须对应一个空的数组[约定]
2. appID: 公司开发者账号的小组ID.包名id[由前缀和ID两部分组成,可以登录苹果开发者网站，在“Certificates, Identifiers & Profiles”页面选择“Identifiers”中选择对应的App ID查看]
3. paths: paths是在项目中的的.entitlements文件中域名后支持的路径，*表示全路径，download表示download路径下的所有url都可以进入到app中打开，其他的路径是不会自动打开应用的(仍然在网页中浏览)

这个JSON格式的文件是app第一次安装，它会从 <https://domain.com/apple-app-site-association> 下载这个文件.下一次在浏览对应的域名时会直接触发打开原生应用.
