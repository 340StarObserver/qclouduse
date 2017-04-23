    mkfs.ext3 /dev/vdb
        // 之前用 df -Th 查看到原来的"/"目录是ext3的
        
        
    fdisk /dev/vdb
        // 建一个大的主分区
        // 依次 : n -> p -> 默认分区编号 -> 默认起始位置 -> 默认结束位置 -> w
        
        
    mkdir /mydata
    mount -t ext3 /dev/vdb /mydata
        // 临时挂载到"/mydata"目录下
        // 注意，如果有多个分区，则需要这样写 :
        //      mount -t ext3 /dev/vdb1 /mydata/p1
        //      mount -t ext3 /dev/vdb2 /mydata/p2
        
        
    vi /etc/fstab
        // 添加一行 /dev/vdb    /mydata     ext3    defaults    0 1
        // 之后重启
        // 注意，如果有多个分区，则需要这样写 :
        //      /dev/vdb1   /mydata/p1  ext3    defaults    0 1
        //      /dev/vdb2   /mydata/p2  ext3    defaults    0 1
