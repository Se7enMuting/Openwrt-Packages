
#### Lean的Openwrt源码仓库 自编译说明

如何编译自己需要的 OpenWrt 固件
-
注意：
-
1. **不**要用 **root** 用户进行编译！！！
2. 国内用户编译前最好准备好梯子
3. 默认登陆IP 192.168.1.1 密码 password


首次编译命令如下:
-
1. 首先装好 Ubuntu 64bit，推荐 Ubuntu 20.04 LTS x64 （虚拟机推荐硬盘大小50G）

2. 命令行输入 `sudo apt-get update` ，然后输入
   `
   sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync
   `

3. 使用 `git clone https://github.com/coolsnowwolf/lede` 命令下载好源代码，然后 `cd lede` 进入目录

4. ```bash
   ./scripts/feeds update -a
   ./scripts/feeds install -a
   make menuconfig
   ```

5. `make -j8 download V=s` 下载dl库（国内请尽量全局科学上网）

6. 输入 `make -j1 V=s` （-j1 后面是线程数。第一次编译推荐用单线程）即可开始编译你要的固件了。

本套代码保证肯定可以编译成功。里面包括了 R21 所有源代码，包括 IPK 的。

=

二次编译（更新Lean源码+更新package）：
```bash
cd lede
git pull
./scripts/feeds update -a && ./scripts/feeds install -a
make defconfig
make -j8 download
make -j$(($(nproc) + 1)) V=s
```

如果需要重新配置：
```bash
cd lede
rm -rf ./tmp && rm -rf .config
make menuconfig
make -j$(($(nproc) + 1)) V=s
```

如果更新Lean源码+更新package+需要重新配置：
```bash
cd lede
git pull
./scripts/feeds update -a && ./scripts/feeds install -a
rm -rf ./tmp && rm -rf .config
make menuconfig
make -j$(($(nproc) + 1)) V=s
```

编译完成后输出路径：bin/targets

#### 本package搬运自如下：

* https://github.com/kenzok8/openwrt-packages


* https://github.com/kenzok8/small


* https://github.com/sirpdboy/sirpdboy-package

#### 使用方法：

 1、 添加下面代码到 openwrt 或lede源码根目录feeds.conf.default文件

```bash
 src-git Se7en https://github.com/Se7enMuting/openwrt-packages
```

 2、 添加passwall依赖

 ```bash
 src-git small https://github.com/kenzok8/small
 ```

- openwrt 固件编译自定义主题与软件
- 来自kenzok8：
- luci-app-openclash       ------------------openclash图形
- luci-app-passwall        ------------------Lienol大神
- luci-app-koolddns---------------------KOOL域名DNS解析工具          
- luci-theme-atmaterial_new  ------------------atmaterial 三合一主题（适配18.06）     
- luci-theme-argon_new     ------------------二合蓝 紫主题
- 来自sirpdboy：
- luci-app-advanced---------------------系统高级设置【自带文件管理功能】
- luci-app-control-speedlimit-----------网速限制
- luci-app-control-timewol--------------定时唤醒
- luci-app-control-weburl---------------管控过滤[集成上网时间控制，黑白名单IP过滤，网址过滤几大功能]
- luci-app-netdata----------------------网络监控中文版
- luci-app-netspeedtest-----------------网络速度测试
- luci-app-wolplus----------------------网络唤醒+
- luci-app-wrtbwmon---------------------带宽监控
- luci-theme-opentopd-------------------opentopd（适配18.06）
- 关机功能插件 : https://github.com/sirpdboy/luci-app-poweroffdevice
- 说明：netdata和wrtbwmon需要手动替换掉lean/package/lean内同文件（删除无效）

#### 开启IPV6
- 选上extra packages——ipv6helper
- 在 Network – Firewall – ip6tables 下启用 ip6tables-extra 和 ip6tables-mod-nat 项。

#### 取消samba
- 取消extra packages——autosamba
- 在 LuCI-Applications里，取消 luci-app-samba

#### PVE安装指令
- qm importdisk 121 /var/lib/vz/template/iso/openwrt-x86-64-generic-squashfs-combined-efi.img local-lvm

#### 20211001版 自选LuCI-App总数：23

#### Lan IP地址修改
- vi /etc/config/network
- i（修改插入）
- 修改完按ESC退出编辑模式
- :wq（保存退出）
