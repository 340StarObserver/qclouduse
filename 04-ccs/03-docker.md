本小结内容 :

        1. 练手 Dockerfile for http+redis
        2. 多进程Docker存在的问题


#### 一.　容器所使用的代码文件 ####

        // 1. 简易的webserver，供读写键值对
        //    curl -XGET  ip:port/get -d 'key=xxx'
        //    curl -XPOST ip:port/set -d 'key=xxx&value=yyy'
        // 2. 代码不严谨，仅仅为了学习docker
        
```python
#!/usr/bin/python
# -*- coding:utf-8 -*-

# Author 	: 	Lv Yang
# Create 	: 	09 June 2017
# Modify 	: 	09 June 2017
# Version 	: 	1.0

import flask
import redis

App = flask.Flask(__name__)
App.secret_key = '\r\x9d1\xd1\xccW\x9e\xa6\x9a\x97[\xb1=\x93\x87\x15s<\xe8\xe3\x13DL?'

def getConn():
	conn = redis.Redis(host = '127.0.0.1', port = 6379, password = '12345678')
	return conn

@App.route('/get', methods = ['GET'])
def get():
	data = flask.request.form
	key = data['key']
	conn = getConn()
	value = conn.get(key)
	if value is None:
		response = {'result' : False}
	else:
		response = {'result' : True, 'value' : value}
	return flask.jsonify(response)

@App.route('/set', methods = ['POST'])
def set():
	data = flask.request.form
	conn = getConn()
	conn.set(data['key'], data['value'])
	return flask.jsonify({'result' : True})

if __name__ == '__main__':
	App.run(host = '0.0.0.0', port = 8080)
```


#### 二.　一开始我的Dockerfile ####

        # content: flask + redis
        # version: 0.0.1
        
        FROM fedora:20
        
        MAINTAINER wuanjun "lvyang@ippclub.org"
        ENV REFRESHED_AT 2017-06-08
        
        # install system tools
        RUN yum -y install net-tools
        
        # install python packages
        RUN yum -y install python-pip
        RUN pip install flask
        RUN pip install redis
        
        # install redis
        RUN yum -y install redis
        
        # create user for redis
        RUN useradd -d /home/wuanjun -m wuanjun
        
        # create dir for redis
        USER wuanjun
        RUN mkdir /home/wuanjun/redis
        RUN mkdir /home/wuanjun/redis/conf
        RUN mkdir /home/wuanjun/redis/data
        RUN mkdir /home/wuanjun/redis/logs
        RUN mkdir /home/wuanjun/redis/pids
        RUN chmod -R 700 /home/wuanjun/redis
        
        # create conf for redis
        WORKDIR /home/wuanjun/redis/conf
        RUN touch redis.conf
        RUN echo 'bind 127.0.0.1'                             >> redis.conf
        RUN echo 'port 6379'                                  >> redis.conf
        RUN echo 'requirepass 12345678'                       >> redis.conf
        RUN echo 'daemonize yes'                              >> redis.conf
        RUN echo 'pidfile /home/wuanjun/redis/pids/redis.pid' >> redis.conf
        RUN echo 'logfile /home/wuanjun/redis/logs/redis.log' >> redis.conf
        RUN echo 'dir /home/wuanjun/redis/data/'              >> redis.conf
        RUN echo 'appendfilename redis.aof'                   >> redis.conf
        RUN echo 'dbfilename redis.rdb'                       >> redis.conf
        RUN echo 'maxmemory 128mb'                            >> redis.conf
        RUN echo 'maxmemory-policy allkeys-lru'               >> redis.conf
        RUN echo 'auto-aof-rewrite-percentage 80-100'         >> redis.conf
        RUN echo 'tcp-backlog 128'                            >> redis.conf
        RUN echo 'timeout 0'                                  >> redis.conf
        RUN echo 'tcp-keepalive 60'                           >> redis.conf
        RUN echo 'loglevel notice'                            >> redis.conf
        RUN echo 'databases 1'                                >> redis.conf
        RUN echo 'save 900 1'                                 >> redis.conf
        RUN echo 'save 300 10'                                >> redis.conf
        RUN echo 'save 60 10000'                              >> redis.conf
        RUN echo 'stop-writes-on-bgsave-error yes'            >> redis.conf
        RUN echo 'rdbcompression yes'                         >> redis.conf
        RUN echo 'rdbchecksum yes'                            >> redis.conf
        RUN echo 'slave-serve-stale-data yes'                 >> redis.conf
        RUN echo 'slave-read-only yes'                        >> redis.conf
        RUN echo 'repl-diskless-sync no'                      >> redis.conf
        RUN echo 'repl-diskless-sync-delay 5'                 >> redis.conf
        RUN echo 'repl-disable-tcp-nodelay yes'               >> redis.conf
        RUN echo 'slave-priority 100'                         >> redis.conf
        RUN echo 'slowlog-log-slower-than 100'                >> redis.conf
        RUN echo 'slowlog-max-len 1000'                       >> redis.conf
        RUN echo 'notify-keyspace-events ""'                  >> redis.conf
        
        # prepare python web code
        USER root
        RUN mkdir /home/wuanjun/web
        ADD main.py /home/wuanjun/web/main.py
        RUN chown -R wuanjun:wuanjun /home/wuanjun/web
        RUN chmod 700 /home/wuanjun/web
        RUN chmod 600 /home/wuanjun/web/main.py
        
        # prepare shell for start-commands
        USER root
        WORKDIR /home/wuanjun
        RUN touch start.sh
        RUN chown wuanjun:wuanjun start.sh
        RUN chmod 700 start.sh
        RUN echo 'redis-server /home/wuanjun/redis/conf/redis.conf' >> start.sh
        RUN echo 'python web/main.py > web/web.log 2>&1 &'          >> start.sh
        
        # start redis & pythonweb
        USER wuanjun
        EXPOSE 8080
        CMD sh start.sh
        
        #
        # 你可以发现，我想要启动redis & pythonweb
        # 而不管是CMD，还是ENTRYPOINT，都只能带一个命令
        # 所以，我把多个想要启动的服务，放到shell脚本中，这样就不违反CMD和ENTRYPOINT的要求了
        #


#### 三.　出现的问题 ####

        首先我构建镜像 :
            sudo docker -H=127.0.0.1:2375 build -f Dockerfile -t kvserver:v0.0.1 .
            
        然后我构建容器 :
            sudo docker -H=127.0.0.1:2375 run --restart=always --name mykv -d kvserver:v0.0.1
            
        然后我通过下述命令查到容器的地址 :
            sudo docker -H=127.0.0.1:2375 inspect mykv
            // ip = 172.17.0.2
            
        然后我满怀希望地进行测试 :
            curl -i -XPOST 172.17.0.2:8080/set -d 'key=name&value=lvyang'
            // 报错，connection refused
        
        >------------------<
        
        // 我曾如下做过另一种尝试 :
        
        1. 在Dockerfile中，删掉最后的CMD启动命令
        
        2. 重新构建镜像
        
        3. 创建交互式容器 :
            sudo docker -H=127.0.0.1:2375 run -i -t --name mykv kvserver:v0.0.1
        
        4. 进入容器中人工执行脚本来启动redis和pythonweb :
            sh start.sh
        
        5. 然后我开了另外一个终端进行curl的测试，成功了
        
        >------------------<
        
        为什么人工运行脚本可以，而把脚本作为容器的启动命令却不行 ?
        
        在nginx容器的资料中发现，必须要把nginx配置文件里的daemon设置成off，然后在run一个容器的时候加上 -d 参数
        
        也就是说 :
        1. docker 中只能运行前台进程
           如果是后台进程的话，它会触发一次fork，在fork之后，父进程结束，这会让docker认为容器的任务完成了，它就死了
        2. docker run -d 让容器中的前台进程不退出
        
        >------------------<
        
        而我有两个进程，一个redis，一个pythonweb，没法在一个容器里运行两个前台任务啊，因为启动第一个前台进程后就会停住
        即，如何才能在docker中运行多个进程
        即，多进程docker的解决方案又是什么 ?
