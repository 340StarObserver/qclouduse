    // 加了一块硬盘后，先测测看IOPS能到多少


    fio \
        --bs=128k --ioengine=libaio --iodepth=32 --direct=1 \
        --rw=randwrite \
        --time_based --runtime=600  \
        --refill_buffers --norandommap --randrepeat=0 --group_reporting \
        --name=test_rand_w \
        --size=10G \
        --filename=/dev/vdb
        // 测试随机写


    fio \
        --bs=4k --ioengine=libaio --iodepth=1 --direct=1 \
        --rw=randrw \
        --rwmixread=70 \
        --time_based --runtime=600  \
        --refill_buffers --norandommap --randrepeat=0 --group_reporting \
        --name=test_rand_rw \
        --size=10G \
        --filename=/dev/vdb
        // 测试随机读写
