## hadoop namenode在每次开机后会启动失败

报错：org.apache.hadoop.hdfs.server.common.InconsistentFSStateException

原因：hadoop.tmp.dir的位置在重启之后被删除了，系统找不到这个文件夹就无法启动

解决方案：在hdfs-site.xml 配置 datanode 和 namenode 存储目录,或跑一次hadoop namenode -format

------

## 