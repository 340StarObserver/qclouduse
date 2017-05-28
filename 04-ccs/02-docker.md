本小结内容 :

        1. Docker服务端后台进程的启动 & 关闭
        2. Docker存储路径的修改
        3. Docker容器demo的上手
        4. Docker容器内的服务怎么访问


#### 一.　Docker daemon 启动脚本 ####

        #!/bin/sh
        docker daemon \
            -H tcp://127.0.0.1:2375 \
            -H unix:///usr/local/docker/run/docker.sock \
            -g /usr/local/docker/data \
            -p /usr/local/docker/run/docker.pid \
            >/usr/local/docker/log/docker.log 2>&1 &
        
        // 1. -H 指定监听地址和端口
        // 2. -H 指定套接字路径
        // 3. -g 指定数据存储路径
                 注意，默认是 /var/lib/docker，但 /var 分区一般不大，故不推荐使用默认值
                 注意，实际企业内部的docker服务器，数据目录建议搭在LVM上
        // 4. -p 指定进程文件路径


#### 二.　Docker daemon 关闭 ####

        > ps -aux | grep docker
        root 27559 <省略几个字段> docker daemon -H tcp://127.0.0.1:2375 -H unix:///usr/local/docker/run/docker.sock
        root 27942 <省略几个字段> grep --color=auto docker
        
        > kill 27559


#### 三.　Docker 服务器的信息 ####

        1. 查看 docker 配置信息
           > sudo docker -H=127.0.0.1:2375 info
           Containers: 0
           Images: 0
           Server Version: 1.9.1
           Storage Driver: aufs
           Root Dir: /usr/local/docker/data/aufs
           Backing Filesystem: extfs
           Dirs: 0
           Dirperm1 Supported: true
           Execution Driver: native-0.2
           Logging Driver: json-file
           Kernel Version: 4.2.0-30-generic
           Operating System: Ubuntu 15.10
           CPUs: 4
           Total Memory: 7.667 GiB
           Name: localhost
           ID: ZZXY:NRMT:CSM7:P5CM:IULR:IDTI:WLAZ:XAJF:IGOZ:4PL4:XR3X:32SQ
           WARNING: No swap limit support
           // 看到 Root Dir 已经不是默认值了
        
        2. 查看 docker 镜像列表
           > sudo docker -H=127.0.0.1:2375 images
           REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
           // 此时没有镜像
        
        3. 查看 docker 容器列表
           > sudo docker -H=127.0.0.1:2375 ps -a
           CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
           // 此时没有容器


#### 四.　Docker 尝试创建第一个镜像 ####

        1. 准备目录
           helloworld
           ├── Dockerfile
           └── server.js
        
        2. 内容 server.js
```js
       var http = require('http');
       var handler = function(request, response) {
           console.log('Received request for URL : ' + request.url);
           response.writeHead(200);
           response.end('Hello World\n');
       };
       var server = http.createServer(handler);
       server.listen(8080, '0.0.0.0');
       // 一个简易web服务
       // 注意，地址要暴露到容器外，这样才可以从容器外面去访问这个服务
```
        
        3. 内容 Dockerfile
           FROM node:6.10.0
           EXPOSE 8080
           COPY server.js .
           CMD node server.js
           // Dockerfile是构建镜像的基础，它包含了 :
           // 1. 所需的运行环境和依赖
           // 2. 对外的输入输出端口
           // 3. 所需的代码文件，配置文件，数据文件等
           // 4. 启动命令
        
        4. 构建镜像
           sudo docker -H=127.0.0.1:2375 build -f Dockerfile -t helloworld:v1
           // 注意，若提示"docker build requires 1 argument"，则在命令最后空格加个.

        5. 查看本地镜像列表
           > sudo docker -H=127.0.0.1:2375 images
           REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
           helloworld          v1                  26da857eec3f        About a minute ago   659.2 MB
           node                6.10.0              a3be0711fad0        12 weeks ago         659.2 MB


#### 五.　Docker 基于镜像创建容器 ####

        sudo docker -H=127.0.0.1:2375 run \
            --restart=always \
            --name helloserver 
            -d helloworld:v1
            
        // 1. 指定该容器自动重启
        // 2. 指定容器名称
        // 3. 指定基于 helloworld:v1 这份镜像
              注意，镜像要使用全称，版本号也要加上，即 repository:tag
              注意，若本地没有这份镜像，则会从官网拉下来
        // 4. -d 表示该容器是后台的


#### 六.　Docker 容器内的服务如何访问 ####

        法一.　通过向容器传达一个任务
        
            sudo docker -H=127.0.0.1:2375 exec -t -i helloserver curl -i -XGET 127.0.0.1:8080
            // -t 表示该任务有一个终端
            // -i 表示该任务接收标准输入
            //    这两点构成了交互式任务的基本条件
            
            // 注意，这种方式，对容器里的服务的监听地址没有要求，即'127.0.0.1'也是可以的

        法二.　通过容器网络地址来访问
        
            sudo docker -H=127.0.0.1:2375 inspect helloserver
            // 查看该容器的详细信息
            // 注意结果中的 IPAddress 字段
            
            curl -i -XGET 172.17.0.2:8080
            // 其中 172.17.0.2 就是该容器的容器网络地址，即上述结果中的IPAddress
            
            // 注意，这种方式，要求容器里的服务的监听地址要是能从外部访问的
