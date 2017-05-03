在多块硬盘之上，创建LVM，可以动态扩容 :

    1. 动态添加新的硬盘到LVM中
    2. 动态增加某分区的容量
    // 两者都是对上层(例如DB，数据库的数据存储在LVM上就比较好)是透明的


#### 一. 单硬盘创建初始的LVM ####

        // 假设我一开始有 /dev/vdb 这块硬盘
        
        pvcreate /dev/vdb
            // 创建第一个物理卷
            // 注意，若该硬盘原来分过区，则需要先格式化
            // 注意，可以通过 lvmdiskscan 或 pvs 来查看物理卷的情况
        
        vgcreate lvm_vg01 /dev/vdb
            // 创建卷组
            // 注意，可以通过 vgs 来查看卷组的情况
        
        lvcreate -L 8G -n lvm_lv01 lvm_vg01
            // 创建逻辑卷
            // 该逻辑卷的初始大小是 8G
            // 该逻辑卷的名称是　　 lvm_lv01
            // 该逻辑卷的所属卷组是 lvm_vg01
        
        mkfs.ext3 /dev/lvm_vg01/lvm_lv01
            // 在　lvm_vg01 这个卷组的 lvm_lv01 的逻辑卷上创建文件系统
        
        mount -t ext3 /dev/lvm_vg01/lvm_lv01 /mydata
            // 挂载


#### 二. 添加新硬盘，并扩容分区 ####

        // 假设现在有 /dev/vdc 这块硬盘可以添加进LVM
        
        pvcreate /dev/vdc
            // 对于要新加入进LVM的硬盘，也要为其创建物理卷
        
        vgextend lvm_vg01 /dev/vdc
            // 把建过物理卷的这块新的硬盘，添加到卷组中
        
        lvextend -L +8G /dev/lvm_vg01/lvm_lv01
            // 为原来的逻辑卷，增加8G容量
        
        resize2fs /dev/lvm_vg01/lvm_lv01
            // resize该逻辑卷的文件系统
