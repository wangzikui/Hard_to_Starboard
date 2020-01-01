## 配置ss

S1:买个国外或香港服务器节点

S2:安装SSR服务器端脚本

```shell
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
chmod +x shadowsocksR.sh
./shadowsocksR.sh
```

S3:配置user、password、端口等,确保端口打开

S4:本机安装SSR软件,按服务器配置来设定(mac版已存在同级文件夹下)

S5:其他信息

```
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status

配置文件路径：/etc/shadowsocks.json
日志文件路径：/var/log/shadowsocks.log
代码安装目录：/usr/local/shadowsocks
```

------

## 