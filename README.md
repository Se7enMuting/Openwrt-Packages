
#### Lean的Openwrt源码仓库 自编译说明

如何编译自己需要的 OpenWrt 固件
--
注意：
--
1. **不**要用 **root** 用户进行编译！！！
2. 国内用户编译前最好准备好梯子
3. 默认登陆IP 192.168.1.1 密码 password


首次编译命令如下:
--
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

--

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
```
https://github.com/kenzok8/openwrt-packages
https://github.com/kenzok8/small
https://github.com/sirpdboy/sirpdboy-package
```
#### 使用方法：

 1. 添加下面代码到 openwrt 或lede源码根目录feeds.conf.default文件
```bash
 src-git Se7en https://github.com/Se7enMuting/openwrt-packages
```

 2. 添加passwall依赖
 ```bash
 src-git small https://github.com/kenzok8/small
 ```

3. openwrt 固件编译自定义主题与软件

- 来自kenzok8：
- luci-app-openclash       ------------------openclash图形
- luci-app-passwall        ------------------Lienol大神          
- luci-theme-atmaterial_new  ------------------atmaterial 三合一主题（适配18.06）     
- luci-theme-argon_new     ------------------二合蓝 紫主题
- 来自sirpdboy：
- luci-app-advanced---------------------系统高级设置【自带文件管理功能】
- luci-app-koolddns---------------------KOOL域名DNS解析工具
- luci-app-aliddns----------------------腾讯DDNS
- luci-app-control-speedlimit-----------网速限制
- luci-app-control-timewol--------------定时唤醒
- luci-app-control-weburl---------------管控过滤[集成上网时间控制，黑白名单IP过滤，网址过滤几大功能]
- luci-app-netdata----------------------网络监控中文版
- luci-app-netspeedtest-----------------网络速度测试
- luci-app-wolplus----------------------网络唤醒+（需要配合control-weburl一起用）
- luci-app-wrtbwmon---------------------带宽监控
- luci-theme-opentopd-------------------opentopd（适配18.06）
- 关机功能插件 : https://github.com/sirpdboy/luci-app-poweroffdevice
- 说明：netdata和wrtbwmon需要手动替换掉lean/package/lean内同文件（删除无效），不然会安装成Lean原版的插件（`./scripts/feeds install -a -f`可强制安装feeds里的插件）
- 自己修改:
- luci-app-tencentddns------------------腾讯DDNS ([官方版本小修改界面](https://github.com/Tencent-Cloud-Plugins/tencentcloud-openwrt-plugin-ddns))

#### 开启IPV6
- 选上extra packages——ipv6helper
- 在 Network – Firewall – ip6tables 下启用 ip6tables-extra 和 ip6tables-mod-nat 项。

#### 取消samba
- 取消extra packages——autosamba
- 在 LuCI-Applications里，取消 luci-app-samba

#### 编译丰富插件时，建议修改下面两项默认大小，留足插件空间。
- Target Images ---> (16) Kernel partition size (in MB)            #默认是 (16) 建议修改 (16-128)
- Target Images ---> (160) Root filesystem partition size (in MB)  #默认是 (160) 建议修改 (512+)

#### 20211001版 自选LuCI-App总数：23+1

#### 编译前更改LAN口的默认IP地址
 ```bash
cd lede
vim package/base-files/files/bin/config_generate
```
#### PVE安装指令
 ```bash
qm importdisk 121 /var/lib/vz/template/iso/openwrt-x86-64-generic-squashfs-combined-efi.img local-lvm
```

#### OpenWrt /LEDE 中安装QEMU Guest Agent
ssh进openwrt，安装qemu-ga 即可
```bash
opkg update
opkg install qemu-ga
reboot
```

#### Lan IP地址修改
- vi /etc/config/network
- i（插入修改）
- 修改完，按ESC退出编辑模式
- :wq（保存退出）

#### 单独编译 OpenWRT ipk 插件
###### 保存插件源码
- 下载源码`git clone`到~`/lede/package`
- 或者已经在feeds里面了，无需在手动下载，`./scripts/feeds update -a && ./scripts/feeds install -a`即可
###### 配置
- `make menuconfig`
- 然后进入对应的子菜单中找到对应插件按`M`表示选中插件，但不编译进固件
###### 编译
- `make package/xxxxx/compile V=99`
- xxxxx 就是你需要单独编译的程序名称
- 注意这里是固定格式，不用填写插件所在的路径，直接名字即可，只要上面配置时选中了
###### ipk 生成路径
- `~/lede/bin/packages/x86_64/xxxx`
###### 其他
- 若无法编译出插件，手动删除`bin`,`feeds`,`package/feeds`这些文件夹，再`./scripts/feeds update -a && ./scripts/feeds install -a`下

#### openwrt源修改，注意要和linux core的版本对应
```
src/gz openwrt_core https://mirrors.cloud.tencent.com/lede/releases/21.02.0/targets/x86/64/packages
src/gz openwrt_base https://mirrors.cloud.tencent.com/lede/releases/21.02.0/packages/x86_64/base
src/gz openwrt_luci https://mirrors.cloud.tencent.com/lede/releases/21.02.0/packages/x86_64/luci
src/gz openwrt_packages https://mirrors.cloud.tencent.com/lede/releases/21.02.0/packages/x86_64/packages
src/gz openwrt_routing https://mirrors.cloud.tencent.com/lede/releases/21.02.0/packages/x86_64/routing
src/gz openwrt_telephony https://mirrors.cloud.tencent.com/lede/releases/21.02.0/packages/x86_64/telephony
```
如果是第三方编译固件，会签名错误，把这段用#注释掉`# option check_signature`
