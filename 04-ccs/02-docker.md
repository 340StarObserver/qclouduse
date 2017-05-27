本小结内容 :

        1. Docker服务端后台进程的启动 & 关闭
        2. Docker存储路径的修改
        3. 上手一个容器demo


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
