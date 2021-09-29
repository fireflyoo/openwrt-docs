先说一下我的硬件架构：矿渣-小娱C5，淘宝买的，大概120左右。
之前用的是[GaryPang][1]的固件，不过现在已经被作者废弃了
所以我转用官方固件，[预览版RC3][2]
卖家已经帮我刷上刷不死后台Breed，进入的办法好像是按着RESET键的同时插上电源等待10秒开机..
设好密码，改好时区。
嗯，第一个问题是因为我演示用的路由是旁路由模式，挂在路由器下面的设备都能正常上网，就旁路由本身无法上网。。
这个先解决再回来更新..
好了，手工加了个DNS Server勉强能用了..(但获取不了IPv6），先这样
下一个问题是矿渣小娱的内置空间太小了,只有25MB。
所以得扩展空间。手头有个闲置的垃圾移动硬盘，插在矿渣的USB口勉强用用看

```bash
opkg update
opkg install kmod-usb-storage
opkg install blkid
#安装网页端界面硬盘挂载交互界面，可能需要重启才会显示出菜单
opkg install block-mount

#这里选用f2fs文件系统
opkg install f2fs-tools
opkg install kmod-fs-f2fs
#格式化前记得分区分区可以用cfdisk
mkfs.f2fs /dev/sda1
```
看下/dev/sd*有了没
有了就去网页界面中的mount point菜单把U盘/硬盘挂起来..这里我选挂载为overlay..
然后重启路由器就进入一个已经扩容的新系统了。
然后装lighttpd+php
```
lighttpd
lighttpd-mod-fastcgi
php8-fpm
php8-mod-ctype
php8-mod-pdo-sqlite
php8-mod-session
php8-mod-tokenizer
php8-mod-filter
zoneinfo-asia
unzip
```
接下来装typecho
```
mkdir /opt
wget https://github.com/typecho/typecho/archive/refs/heads/master.zip
unzip master -d /opt
mv /opt/typecho-master /opt/www
chown http:www-data /opt/www -R
```
接下来改配置文件/etc/lighttpd/lighttpd.conf
增加这段
```
fastcgi.server += (
    ".php" => (
        "localhost" => (
            "socket" => "/var/run/php8-fpm.sock"
    )
  )
)
$SERVER["socket"] == "[::]:80" {}

```
改/etc/php.ini
注释或删掉以下这段
```
;doc_root= "/www"
```
改/etc/php8-fpm.d/www.conf
```
user=http
group=www-data
listen.owner = http
listen.group = www-data
```
重启应用
```
/etc/init.d/lighttpd restart
/etc/init.d/php8-fpm restart
```
进入网页，能成功打开typecho安装页面就成功了
安装页面里的
网站网址填
```
/
```
其他随意
  [1]: https://op.supes.top/firmware/obsolete/XY-C5/05.26-openwrt-ramips-mt7621-xy-c5-squashfs-sysupgrade.bin
  [2]: https://downloads.openwrt.org/releases/21.02.0-rc3/targets/ramips/mt7621/openwrt-21.02.0-rc3-ramips-mt7621-xiaoyu_xy-c5-squashfs-sysupgrade.bin
