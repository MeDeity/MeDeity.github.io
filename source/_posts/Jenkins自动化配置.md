##### Docker安装Jenkins

```shell
#拉取镜像
docker pull jenkins/jenkins:lts
# 创建外部目录
mkdir -p /home/jenkins_home
# 运行容器
docker run -d --name jenkins -p 8181:8080 -v /home/jenkins_home:/home/jenkins_home jenkins/jenkins:lts
# 进入容器内部
docker exec -it jenkins /bin/bash
# 获取初始密码
cat /var/jenkins_home/secrets/initialAdminPassword
```

> 特别注意:如果访问不了,使用的是阿里云的服务器器,请确保在安全组放行特定的端口

##### 插件安装

除了安装推荐的插件外,由于需要部署的是maven项目,需要另行安装
-Maven Integration plugin
由于jenkins与应用容器不在同一台服务器上,需另行安装
-Publish Over SSH

##### Publish Over SSH配置

在系统管理/配置/Publish over SSH下新增配置

1. Name: 应用服务器的名称
2. Hostname: 主机地址
3. Username: 登录用户名
4. Remote Directory: 远程目录,这里我直接使用根目录【/】
勾选Use password authentication, or use a different key,直接使用密码登录
在 Passphrase / Password 处直接输入密码,然后点击Test Configuration,直至显示Success.
则Publish Over SSH配置配置成功

##### JDK配置

在系统管理/全局工具配置/JDK安装
如果使用的是外部的JDK,根据我们设置的volume(/home/jenkins_home),需要将JDK安装在此处,否则Jenkins会找不到,当然在/home/jenkins_home
处软链接JDK的位置应该也可以,具体没试验.
JDK别名:JDK(随你高兴)
JAVA_HOME: /home/jenkins_home/java/jdk1.8.0_241
如果你对JDK版本没有特殊要求,你也可以让jenkins自行安装

##### Maven配置

同JDK配置.尽量在volume(/home/jenkins_home)目录下,要不然会出现找不到Maven的情况.当然最省事的就是让Jenkins自行安装

##### Maven 任务

###### General选项卡

在描述中填写该构建任务的备注信息,其中特别重要的是请务必开启丢弃旧的构建,在多次构建后,很容易将服务器撑爆.一般情况下可以设置保留几天内的构建,并保留最大的构建数.

###### 源码管理选项卡

推荐使用GIT进行项目管理.添加必要的认证证书,指定构建分支(一般情况下使用*/master)

###### 构建触发器选项卡

选中Build whenever a SNAPSHOT dependency is built及GitHub hook trigger for GITScm polling
其中GitHub hook trigger for GITScm polling是个比较有用的一个功能

###### 构建环境选项卡

勾选 Add timestamps to the Console Output 将构建时间记录现在在控制台输出中

###### Build选项卡

Root POM: pom.xml
Goals and options: clean install -Ptest -Dmaven.test.skip=true

>跳过测试用例

###### Post Steps选项卡

1. 选中 Run only if build succeeds
2. 在Add post-build step 中选中(Send files or execute commands over SSH)
3. SSH Publishers设置
4. Name: 此处应该有下拉选项(如果没有,说明你没有配置,请返回到 Publish Over SSH配置 进行配置) 选择应用服务器
Transfers Set 设置
5. Source files: 需要上传到服务器的文件,多个文件可以使用逗号分隔
6. Remove prefix: 删除部分前缀 例如你的源文件的目录为:target/deployment/images/**/,但是你不想创建 target/deployment 这两层目录.
在此处就可以配置 target/deployment,这样 在远程服务器上就只会生成 images文件夹目录.
7. Remote directory: 文件将拷贝到哪个目录下,如果该目录不存在会自动创建
8. Exec command: 拷贝完成后在远程服务器上执行的脚本

```
cd /home/project/license_process/
tar -zxvf license_process-1.0-SNAPSHOT-bin.tar.gz
chmod -R 777 license_process-1.0-SNAPSHOT
cd license_process-1.0-SNAPSHOT/bin
./auto.sh stop
./auto.sh start
```

##### auto 相关的shell脚本

```shell
#!/bin/sh
#
#该脚本为Linux下启动java程序的通用脚本。即可以作为开机自启动service脚本被调用，
#也可以作为启动java程序的独立脚本来使用。
#
#JDK所在路径
JAVA_HOME="/usr/local/java/jdk1.8.0_241"

#Java程序所在的目录（classes的上一级目录）
APP_HOME=/home/project/projectName

#需要启动的Java主程序（main方法类）
APP_MAINCLASS=com.*.XXXApplication

#拼凑完整的classpath参数，包括指定lib目录下所有的jar
CLASSPATH=$APP_HOME/classes
for i in "$APP_HOME"/lib/*.jar; do
   CLASSPATH="$CLASSPATH":"$i"
done

#java虚拟机启动参数
JAVA_OPTS="-ms512m -mx512m -Xmn128m -Djava.awt.headless=true -XX:MaxPermSize=64m"

###################################
#(函数)判断程序是否已启动
#
#说明：
#使用JDK自带的JPS命令及grep命令组合，准确查找pid
#jps 加 l 参数，表示显示java的完整包路径
#使用awk，分割出pid ($1部分)，及Java程序名称($2部分)
#当jps命令不可用时,使用: ps -ef | grep $APP_MAINCLASS | grep -v "grep" | awk '{print $2}' 代替
###################################
#初始化psid变量（全局）
psid=0

checkpid() {
   javaps=`$JAVA_HOME/bin/jps -l | grep $APP_MAINCLASS`
   #javaps=`ps -ef | grep $APP_MAINCLASS | grep -v "grep" | awk '{print $2}'`

   if [ -n "$javaps" ]; then
      psid=`echo $javaps | awk '{print $1}'`
   else
      psid=0
   fi
}

###################################
#(函数)启动程序
#
#说明：
#1. 首先调用checkpid函数，刷新$psid全局变量
#2. 如果程序已经启动（$psid不等于0），则提示程序已启动
#3. 如果程序没有被启动，则执行启动命令行
#4. 启动命令执行后，再次调用checkpid函数
#5. 如果步骤4的结果能够确认程序的pid,则打印[OK]，否则打印[Failed]
#注意：echo -n 表示打印字符后，不换行
#注意: "nohup 某命令 >/dev/null 2>&1 &" 的用法
###################################
start() {
   checkpid

   if [ $psid -ne 0 ]; then
      echo "================================"
      echo "warn: $APP_MAINCLASS already started! (pid=$psid)"
      echo "================================"
   else
      echo -n "Starting $APP_MAINCLASS ..."
      # -DlogFn=active 指的是生产日志文件名为active
      nohup $JAVA_HOME/bin/java $JAVA_OPTS -classpath $CLASSPATH $APP_MAINCLASS >/dev/null 2>nohup.out &
      checkpid
      if [ $psid -ne 0 ]; then
         echo "(pid=$psid) [OK]"
      else
         echo "[Failed]"
      fi
   fi
}

###################################
#(函数)停止程序
#
#说明：
#1. 首先调用checkpid函数，刷新$psid全局变量
#2. 如果程序已经启动（$psid不等于0），则开始执行停止，否则，提示程序未运行
#3. 使用kill -9 pid命令进行强制杀死进程
#4. 执行kill命令行紧接其后，马上查看上一句命令的返回值: $?
#5. 如果步骤4的结果$?等于0,则打印[OK]，否则打印[Failed]
#6. 为了防止java程序被启动多次，这里增加反复检查进程，反复杀死的处理（递归调用stop）。
#注意：echo -n 表示打印字符后，不换行
#注意: 在shell编程中，"$?" 表示上一句命令或者一个函数的返回值
###################################
stop() {
   checkpid

   if [ $psid -ne 0 ]; then
      echo -n "Stopping $APP_MAINCLASS ...(pid=$psid) "
      kill -9 $psid
      if [ $? -eq 0 ]; then
         echo "[OK]"
      else
         echo "[Failed]"
      fi

      checkpid
      if [ $psid -ne 0 ]; then
         stop
      fi
   else
      echo "================================"
      echo "warn: $APP_MAINCLASS is not running"
      echo "================================"
   fi
}

###################################
#(函数)检查程序运行状态
#
#说明：
#1. 首先调用checkpid函数，刷新$psid全局变量
#2. 如果程序已经启动（$psid不等于0），则提示正在运行并表示出pid
#3. 否则，提示程序未运行
###################################
status() {
   checkpid

   if [ $psid -ne 0 ];  then
      echo "$APP_MAINCLASS is running! (pid=$psid)"
   else
      echo "$APP_MAINCLASS is not running"
   fi
}

###################################
#(函数)打印系统环境参数
###################################
info() {
   echo "System Information:"
   echo "****************************"
   echo `head -n 1 /etc/issue`
   echo `uname -a`
   echo
   echo "JAVA_HOME=$JAVA_HOME"
   echo `$JAVA_HOME/bin/java -version`
   echo
   echo "APP_HOME=$APP_HOME"
   echo "APP_MAINCLASS=$APP_MAINCLASS"
   echo "****************************"
}

###################################
#读取脚本的第一个参数($1)，进行判断
#参数取值范围：{start|stop|restart|status|info}
#如参数不在指定范围之内，则打印帮助信息
###################################
case "$1" in
   'start')
      start
      ;;
   'stop')
     stop
     ;;
   'restart')
     stop
     start
     ;;
   'status')
     status
     ;;
   'info')
     info
     ;;
  *)
     echo "Usage: $0 {start|stop|restart|status|info}"
     exit 1
esac
exit 0
```

至此,构建任务完成,点击开始构建应该就能正常运行

[Docker 搭建 Jenkins 实现自动部署](https://segmentfault.com/a/1190000021566330?utm_source=sf-related)
[在CentOS 7上搭建 Jenkins + Maven + Git 持续集成环境](https://segmentfault.com/a/1190000017741598)
[jenkins+maven+docker+github全自动化部署SpringBoot实例](https://segmentfault.com/a/1190000014325300)
[Maven属性（properties）标签的使用](https://www.cnblogs.com/EasonJim/p/6815365.html)
