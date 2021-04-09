# iostat


# 磁盘IOPS
## 测试随机写IOPS：
fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/opt/[file] -name=Rand_Write_Testing
## 测试随机读IOPS：
fio -direct=1 -iodepth=128 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/opt/[file] -name=Rand_Read_Testing
## 测试写吞吐量：
fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/opt/[file] -name=Write_PPS_Testing
## 测试读吞吐量：
fio -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/opt/[file] -name=Read_PPS_Testing