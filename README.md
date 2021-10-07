
# 2021-10-06 Lean版本Openwrt（[R21.10.1](https://github.com/coolsnowwolf/lede/tree/687407acdc585355acd24726eac61dca60cd06fb)）源码仓库+passwalll+openclash，自编译onebyone说明

### 注意：
1. **不**要用 **root** 用户进行编译！！！
2. 国内用户编译前最好准备好梯子
3. 默认登陆IP 192.168.1.* （后面有修改），密码 password

## 1.开始编译流程：

#### 首次编译
1. 首先用VMware Workstation Pro 16 装好 Ubuntu 20.04 LTS x64 （虚拟机推荐硬盘大小50G-100G）

2. 命令行输入 `sudo apt-get update` ，然后输入`sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync`

3. 使用 `git clone https://github.com/coolsnowwolf/lede.git` 命令下载好源代码，然后 `cd lede` 进入目录

4. `git reset --hard 687407acdc585355acd24726eac61dca60cd06fb`  退回R21.10.1版本

5. 更改LAN口的默认IP地址
	```
	cd lede
	vim package/base-files/files/bin/config_generate
	vi /etc/config/network
	i                                                 #插入模式；找到192.168.1.1，修改；按ESC退出编辑模式
	:wq                                               #保存退出
	```

6. 添加下面代码到lede源码根目录`feeds.conf.default`文件（添加passwall和自定义的feeds源）
	```
	src-git lienol https://github.com/xiaorouji/openwrt-passwall
	src-git Se7en https://github.com/Se7enMuting/openwrt-packages
	```

7. 添加openclash源
	```
	# cd进入Clone项目
	mkdir package/luci-app-openclash
	cd package/luci-app-openclash
	git init
	git remote add -f origin https://github.com/vernesong/OpenClash.git
	git config core.sparsecheckout true
	echo "luci-app-openclash" >> .git/info/sparse-checkout
	git pull --depth 1 origin master
	git branch --set-upstream-to=origin/master master

	# 编译 po2lmo (如果有po2lmo可跳过)
	pushd luci-app-openclash/tools/po2lmo
	make && sudo make install
	popd

	# 回退到主项目目录
	cd ../..
	```

8. 添加host/upx依赖，passwall要用，也可以不装，只是会有个warning而已
	```
	git clone https://github.com/kuoruan/openwrt-upx.git package/openwrt-upx
	```

9. 删除lean原包中的luci-app-wrtbwmon和luci-app-netdata，避免warning
	```
	rm -rf package/lean/luci-app-wrtbwmon/
	rm -rf package/lean/luci-app-netdata/
	```

10. update feeds
	```
	./scripts/feeds update -a
	```

11. 强制安装（-f）feeds，如feeds和lean源有同名的package，强制安装feed里的
	```
	./scripts/feeds install -a -f
	```

12. 添加poweroff按钮，这步必须要在feeds install之后，编译之前
	```
	cd lean #进入源码目录
	curl -fsSL https://raw.githubusercontent.com/sirpdboy/other/master/patch/poweroff/poweroff.htm > ./feeds/luci/modules/luci-mod-admin-full/luasrc/view/admin_system/poweroff.htm
	curl -fsSL https://raw.githubusercontent.com/sirpdboy/other/master/patch/poweroff/system.lua > ./feeds/luci/modules/luci-mod-admin-full/luasrc/controller/admin/system.lua
	```

13. 进入交互式配置界面

		make menuconfig

	- 开启IPV6
	   - 选上extra packages——ipv6helper
	   - 在 Network – Firewall – ip6tables 下启用 ip6tables-extra 和 ip6tables-mod-nat 项。
	- 取消samba
	   - 取消extra packages——autosamba
	   - 在 LuCI-Applications里，取消 luci-app-samba
	- 编译丰富插件时，建议修改下面两项默认大小，留足插件空间。
	   - Target Images ---> (16) Kernel partition size (in MB)
   	*（默认是 16，建议修改成 64）*
	   - Target Images ---> (160) Root filesystem partition size (in MB)
   	*（默认是 160，建议修改成 512+）*
	- Base system > dnsmasq-full 选满（HAVE不选）
	- 添加主题
	- luci 选23-2个；首次编译，openclash和passwall先不选
	- 最后再确认kmod-tun被选上了（openclash依赖，一定要最后确认一次，会被自动取消掉）
   	Kernel modules > Network Support > kmod-tun


14. `make -j8 download V=s` 下载dl库（国内请尽量全局科学上网）

15. 输入 `make -j1 V=s` （-j1 后面是线程数；第一次编译推荐用单线程）即可开始编译你要的固件了

16. 编译完成后输出路径：bin/targets

#### 第二次完整编译，带上openclash和passwall
1. 清除配置
	```
	rm -rf ./tmp && rm -rf .config
	```

2. 重复**首次编译**中的第12步，luci里选23个（openclash和passwall）
	```
	make menuconfig
	```

3. 可多线程编译
	```
	make -j$(($(nproc) + 1)) V=s
	```

4. 编译完成后输出路径：bin/targets

#### 更新编译
```
cd lede                                                       #进入LEDE目录
git pull                                                      #同步更新大雕源码
cd package/luci-app-openclash && git pull                     #进入OpenClash目录并更新源码
cd ../..                                                      #退回到lede目录
./scripts/feeds update -a && ./scripts/feeds install -a -f
make menuconfig
make -j$(($(nproc) + 1)) V=s
```

## 2.其他说明：

#### 二次编译（更新Lean源码+更新package）
```
cd lede
git pull
./scripts/feeds update -a && ./scripts/feeds install -a
make defconfig
make -j8 download
make -j$(($(nproc) + 1)) V=s
```

#### 如果需要重新配置
```
cd lede
rm -rf ./tmp && rm -rf .config
make menuconfig
make -j$(($(nproc) + 1)) V=s
```

#### 如果更新Lean源码+更新package+需要重新配置
```
cd lede
git pull
./scripts/feeds update -a && ./scripts/feeds install -a
rm -rf ./tmp && rm -rf .config
make menuconfig
make -j$(($(nproc) + 1)) V=s
```

#### 本package搬运自如下
```
https://github.com/kenzok8/openwrt-packages
https://github.com/kenzok8/small
https://github.com/sirpdboy/sirpdboy-package
https://github.com/xiaorouji/openwrt-passwall
https://github.com/vernesong/OpenClash
https://github.com/Se7enMuting/openwrt-packages
```

#### openwrt 固件编译自定义主题与软件
1. 来自kenzok8：
	- luci-app-openclash       ------------------openclash图形
	- luci-app-passwall        ------------------Lienol大神
	- luci-theme-atmaterial_new  ------------------atmaterial 三合一主题（适配18.06）
	- luci-theme-argon_new     ------------------二合蓝 紫主题
2. 来自sirpdboy：
	- luci-app-advanced---------------------系统高级设置【自带文件管理功能】
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
3. 自己修改:
	- luci-app-tencentddns------------------腾讯DDNS ([官方版本小修改界面](https://github.com/Tencent-Cloud-Plugins/tencentcloud-openwrt-plugin-ddns))

#### PVE安装指令
	qm importdisk 121 /var/lib/vz/template/iso/openwrt-x86-64-generic-squashfs-combined-efi.img local-lvm

#### OpenWrt /LEDE 中安装QEMU Guest Agent

1. ssh进openwrt，安装qemu-ga 即可
	```
	opkg update
	opkg install qemu-ga
	reboot
	```
#### 单独编译 OpenWRT ipk 插件

1. 保存插件源码
	- 下载源码`git clone`到~`/lede/package`
	- 或者已经在feeds里面了，无需在手动下载，`./scripts/feeds update -a && ./scripts/feeds install -a`即可

2. 配置
	- `make menuconfig`
	- 然后进入对应的子菜单中找到对应插件按`M`表示选中插件，但不编译进固件

3. 编译
	- `make package/xxxxx/compile V=99`
	- xxxxx 就是你需要单独编译的程序名称
	- 注意这里是固定格式，不用填写插件所在的路径，直接名字即可，只要上面配置时选中了

4. ipk 生成路径
	- `~/lede/bin/packages/x86_64/xxxx`

5. 其他
	- 若无法编译出插件，手动删除`bin`,`feeds`,`package/feeds`这些文件夹，再`./scripts/feeds update -a && ./scripts/feeds install -a`下

#### menuconfig之后，从配置文件diffconfig导出差异文件seed.config，给云编译备用

	./scripts/diffconfig.sh > seed.config

#### openwrt源修改，注意要和linux core的版本对应
	src/gz openwrt_core https://mirrors.cloud.tencent.com/lede/releases/21.02.0/targets/x86/64/packages
	src/gz openwrt_base https://mirrors.cloud.tencent.com/lede/releases/21.02.0/packages/x86_64/base
	src/gz openwrt_luci https://mirrors.cloud.tencent.com/lede/releases/21.02.0/packages/x86_64/luci
	src/gz openwrt_packages https://mirrors.cloud.tencent.com/lede/releases/21.02.0/packages/x86_64/packages
	src/gz openwrt_routing https://mirrors.cloud.tencent.com/lede/releases/21.02.0/packages/x86_64/routing
	src/gz openwrt_telephony https://mirrors.cloud.tencent.com/lede/releases/21.02.0/packages/x86_64/telephony

如果是第三方编译固件，会签名错误，把这段用#注释掉`# option check_signature`
