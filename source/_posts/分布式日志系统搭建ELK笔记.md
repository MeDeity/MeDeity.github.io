---
title: '分布式日志系统搭建ELK笔记'
date: 2020-03-31
categories: 
  - 运维
tags:
  - 技巧
---
一、下载镜像并运行

```
docker pull elasticsearch:7.7.1
# 获取镜像的ID（IMAGE ID）
docker images
docker run -d -e ES_JAVA_POTS="-Xms512m -Xmx512m"  -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 --name elasticsearch-7.7.1 830a894845e3
```

访问<http://IP-Address:9200>,如果有以下回显,则说明elasticsearch安装成功

```
{
  "name" : "6e7dd102a29a",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "sCcZvGiaQNe3OBQCwcGuoA",
  "version" : {
    "number" : "7.7.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ad56dce891c901a492bb1ee393f12dfff473a423",
    "build_date" : "2020-05-28T16:30:01.040088Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

二、下载镜像并运行

新建kibana.yml配置项

```yml
server.name: kibana
server.host: "0"
# elasticsearch 是link的别名
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
monitoring.ui.container.elasticsearch.enabled: true
# 汉化
i18n.locale: "zh-CN"
```

```
docker pull kibana:7.7.1 
docker run --link 6e7dd102a29a:elasticsearch -v /home/config/kibana.docker.yml:/config/kibana.yml -p 5601:5601 -d  --name kibana7.7.1 6de54f813b39 
```

下载镜像FileBeat并运行

```yml
#=========================== Filebeat inputs ==============
filebeat.inputs:

- type: log
  enabled: true
  ##配置你要收集的日志目录，可以配置多个目录
  paths:
    - /home/project/beautiful-license/license-1.0.0/bin/log/*.log

  ##配置多行日志合并规则，已时间为准，一个时间发生的日志为一个事件      
  multiline.pattern: '^\d{4}-\d{2}-\d{2}'
  multiline.negate: true
  multiline.match: after

## 设置kibana的地址，开始filebeat的可视化  
setup.kibana.host: "http://kibana:5601"
setup.dashboards.enabled: true
#-------------------------- Elasticsearch output ---------
output.elasticsearch:
    hosts: ["http://120.79.243.136:9200"]
    index: "filebeat-%{+yyyy.MM.dd}"

setup.template.name: "my-log"
setup.template.pattern: "my-log-*"
json.keys_under_root: false
json.overwrite_keys: true
##设置解析json格式日志的规则
processors:
  - decode_json_fields:
    fields: [""]
    target: json
```

```
docker run -d -v /home/config/filebeat.docker.yml:/usr/share/filebeat/filebeat.yml -v /home/project/beautiful-license/license-1.0.0/bin/log/:/logs/ --link 6e7dd102a29a:elasticsearch --link e1c6b29bff08:kibana  --name filebeat7.7.1   a4c1bdadf04d


docker run -d -v /home/config/filebeat.docker.yml:/usr/share/filebeat/filebeat.yml --link 6e7dd102a29a:elasticsearch --link e1c6b29bff08:kibana  --name filebeat7.7.1   a4c1bdadf04d
```

[Docker+EFK 快速搭建日志收集系统](https://zhuanlan.zhihu.com/p/147799204)
[docker 安装Filebeat](https://www.cnblogs.com/provence666/p/10781123.html)
[Metricbeat+Elastcsearch+Kibana系统和服务监控-Failed to load directory /data/app/metricbeat/kibana/6/index-pattern](https://www.jianshu.com/p/f0e3a17ade37?from=groupmessage)
[ELK Stack7.2一键多机部署脚本](https://cloud.tencent.com/developer/article/1706326)
[Linux 安装 Elasticsearch](https://wxnacy.com/2018/07/04/linux-install-es/)
[使用 Elasticsearch, Kibana, Filebeat 搭建日志工作站](https://wxnacy.com/2018/07/05/elk/)
