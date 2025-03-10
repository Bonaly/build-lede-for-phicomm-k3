# build-lede-for-phicomm-k3
编译斐讯k3版本的lede，默认极简配置

# 使用步骤
如果你只是想下载lede k3的固件包，直接在Actions中选择最新workflow下载Artifacts即可

默认登陆IP 192.168.2.1 密码 password


如果想自己编译最新lede k3固件包，按下面步骤即可：

正常情况：
- 0. fork本仓库
- 1. 在Actions->All workflows中点击OpenWrt-CI
- 2. 通过Run workflow按钮运行编译
- 3. 编译成功以后在Artifacts下载固件包即可

如果上面步骤编译不成功，则需要自己手动生成一下.config
- 0. fork本仓库
- 1. 在Actions->All workflows中点击OpenWrt-CI
- 2. 通过Run workflow按钮运行编译（编译前把SSH connection to Actions改为true）
- 3. 编译运行到SSH connection to Actions时会出现类似下面信息等待你手动操作
    ```
    To connect to this session copy-n-paste the following into a terminal or browser:
    ssh Y26QeagDtsPXp2mT6me5cnMRd@nyc1.tmate.io
    https://tmate.io/t/Y26QeagDtsPXp2mT6me5cnMRd
    ```
- 4. 复制SSH连接命令粘贴到终端内执行，或者复制链接在浏览器中打开使用网页终端。（网页终端可能会遇到黑屏的情况，按Ctrl+C即可）
- 5. 执行make menuconfig进入配置界面
- 6. 参考[手动编译斐讯K3路由器OpenWRT固件，开启SSRP。极简配置，适合初学者](https://www.youtube.com/watch?v=IPk3q5rbtN8)进行配置，其实主要选择三点：
    - 1. Target选择带BCM53xx ARM的
    - 2. Target Profile选择PHCOMM K3
    - 3. （可选）根据需要在LuCI->Applications中选择额外安装的应用
- 7. 完成后按Ctrl+D组合键或执行exit命令退出，退出时提示是否保存时选择Yes
- 8. 按照正常情况的步骤再次编译即可

# 常用操作
## 推荐配置
- [PLUS+]将你的上网姿势设定为使用本机端口为5335的DNS服务，选择绕过某些IP模式
- [PassW]将你的上网姿势设定为DNS模式-不使用，选择绕过某些IP模式

## Aria2下载配置
- 1. 编译的时候加上aira2：
    ```
    # Aria2 luci配置页面
    CONFIG_PACKAGE_luci-app-aria2=y
    # Aria2核心包
    CONFIG_PACKAGE_aria2=y
    # Aria2 下载管理页面（可选，luci-app-aria2会自带一个Aria-Ng，可以两个都按上，选自己喜欢的用）
    CONFIG_PACKAGE_webui-aria2=y
    ```
- 2. 路由器管理界面开启aira2
    1. 路由器接入硬盘/U盘
    2. 在系统->挂载点找到已经挂载的硬盘，例如`/mnt/sda2`
    3. 进入`Aria2`的`文件和目录`选项，修改`默认下载目录`为你的磁盘挂载路径：`/mnt/sda2`，保存配置
    4. 这时候Aria2就启动了，界面上也出现Aria-Ng和webui-aria2了，点击去就能新建下载任务了

# 关键技术
## .config怎么配置比较合适
配置.config有两个方法：
- make menuconfig:弹出用户交互界面，由根据用户选择产生新的.config
- make defconfig:在已有.config的基础上，补充一些默认值生成.config(可能是这样吧)

一般搜索编译k3的教程，都提供了参考的.config(例如[关于Github Action自动编译Lean_Openwrt的配置修改问题](https://zhuanlan.zhihu.com/p/94527343)里就有提供.config配置方法)，但问题是lede的config经常变化，网上的.config一开始可能可以用，过一段时间就不能用了，兼容性不好。

如果需要使用make menuconfig的话就需要用户交互，在技术上可以通过ssh连接到Github Actions来实现，但是每次ssh连接手动选择再编译比较麻烦，对新手也不友好。

最终采用的方案是做成可配置的方式，使用make menuconfig还是make defconfig由用户自己选择，并且使用make menuconfig生成的.config都提交到当前仓库中，
下次使用make defconfig的时候自动使用当前仓库的.config

所以最终效果是用户可以直接使用make defconfig结合本仓库自带的.config编译，如果失败了，再采用make menuconfig方式编译，同时自动更新.config文件，
下次又可以使用make defconfig方式了，只有在必须的时候才使用make menuconfig，简化编译操作。

## k3screenctrl发展史
[updateing/k3screenctrl](https://github.com/updateing/k3screenctrl)(逆向做出第一版k3screenctrl) -> [RightFS/k3screenctrl](https://github.com/RightFS/k3screenctrl)(提供了屏幕固件的文件) -> [Hill-98/luci-app-k3screenctrl](https://github.com/Hill-98/luci-app-k3screenctrl)(添加了网页设置的支持) -> [zxlhhyccc/Hill-98-k3screenctrl](https://github.com/zxlhhyccc/Hill-98-k3screenctrl)(支持了天气) -> [lwz322/k3screenctrl](https://github.com/lwz322/k3screenctrl)(修复了天气导致屏幕卡死问题)

- [k3screenctrl发展史](https://github.com/lwz322/k3screenctrl#%E5%8F%AF%E4%BB%A5%E5%85%AC%E5%BC%80%E7%9A%84%E6%83%85%E6%8A%A5)
- [替换lede中k3screenctrl的方法](https://github.com/yangxu52/Phicomm-K2P-K3-OpenWrt-Firmware/blob/2efaffe30b1da709ae9c6c881f8462873ac06af5/PhicommK3/lede/part1.sh#L33)

# 参考链接
- [手动编译斐讯K3路由器OpenWRT固件，开启SSRP。极简配置，适合初学者](https://www.youtube.com/watch?v=IPk3q5rbtN8)
- [使用 GitHub Actions 云编译 OpenWrt源码(主要参考，有.config完美生成流程)](https://github.com/P3TERX/Actions-OpenWrt)
- [使用 GitHub Actions 云编译 OpenWrt教程](https://p3terx.com/archives/build-openwrt-with-github-actions.html)
- [OpenWrt CI在线集成自动编译环境(次要参考)](https://github.com/KFERMercer/OpenWrt-CI)
- [OpenWrt-Autobuild](https://github.com/vgist/OpenWrt-Autobuild)
- [Phicomm-K2P-K3-OpenWrt-Firmware](https://github.com/yangxu52/Phicomm-K2P-K3-OpenWrt-Firmware)

- [面向小白的Github_Action使用workflow自动编译lean_openwrt教程](https://zhuanlan.zhihu.com/p/94402324)
- [关于Github Action自动编译Lean_Openwrt的配置修改问题](https://zhuanlan.zhihu.com/p/94527343)

- [OpenWrt官方原版+lede包编译斐讯k3教程](https://gist.github.com/luoqeng/ad535fca4a87a1cfb9a9c1def151b837)

- [helloworld](https://github.com/fw876/helloworld)
- [lede](https://github.com/coolsnowwolf/lede)

# luci-app软件包
luci-app软件包安装方式多种：
- 在make menuconfig的时候选择
- 直接修改本仓库.config文件（想安装哪个就解除哪个注释，不用管它的依赖包，会自动安装的）
- 在路由器管理后台的系统->安装包里安装/卸载

以下是常见luci-app包介绍([出处](https://www.right.com.cn/forum/thread-3682029-1-1.html))
```
 -----------------------------------------------------------------------------------------
LuCI ---> Applications ---> luci-app-accesscontrol  #访问时间控制
LuCI ---> Applications ---> luci-app-adbyby-plus   #广告屏蔽大师Plus +
LuCI ---> Applications ---> luci-app-arpbind  #IP/MAC绑定
LuCI ---> Applications ---> luci-app-autoreboot  #支持计划重启
LuCI ---> Applications ---> luci-app-ddns   #动态域名 DNS（集成阿里DDNS客户端）
LuCI ---> Applications ---> luci-app-filetransfer  #文件传输（可web安装ipk包）
LuCI ---> Applications ---> luci-app-firewall   #添加防火墙
LuCI ---> Applications ---> luci-app-frpc   #内网穿透 Frp
LuCI ---> Applications ---> luci-app-ipsec-vpnd  #VPN服务器 IPSec
LuCI ---> Applications ---> luci-app-nlbwmon   #网络带宽监视器
LuCI ---> Applications ---> luci-app-ramfree  #释放内存
LuCI ---> Applications ---> luci-app-samba   #网络共享（Samba）
-------------------------------------------------------------------------------------------
LuCI ---> Applications ---> luci-app-ssr-plus   #SSR科学上网Plus+
    luci-app-ssr-plus ---> Include Shadowsocks Libev Client  #SS Libev客户端(轻量级)
    luci-app-ssr-plus ---> Include Shadowsocks Libev Server  #SS Libev服务端(轻量级)
    luci-app-ssr-plus ---> Include ShadowsocksR Libev Client  #SSR Libev客户端(轻量级)
    luci-app-ssr-plus ---> Include ShadowsocksR Libev Server  #SSR Libev服务端(轻量级)
    luci-app-ssr-plus ---> Include Include Shadowsocks Simple Obfs Plugin  #SS Simple-Obfs混淆代理(Nginx)
    luci-app-ssr-plus ---> Include Xray  #Xray代理(XTLS)
-------------------------------------------------------------------------------------------
LuCI ---> Applications ---> luci-app-turboacc   #Turbo ACC 网络加速(支持 Fast Path 或者 硬件 NAT)
LuCI ---> Applications ---> luci-app-unblockmusic  #解锁网易云灰色歌曲3合1新版本
LuCI ---> Applications ---> luci-app-upnp   #通用即插即用UPnP（端口自动转发）
LuCI ---> Applications ---> luci-app-vlmcsd  #KMS服务器设置
LuCI ---> Applications ---> luci-app-vsftpd  #FTP服务器
LuCI ---> Applications ---> luci-app-wifischedule  #WiFi 计划
LuCI ---> Applications ---> luci-app-wireless-regdb  #WiFi无线
LuCI ---> Applications ---> luci-app-wol   #WOL网络唤醒
LuCI ---> Applications ---> luci-app-xlnetacc  #迅雷快鸟
LuCI ---> Applications ---> luci-app-zerotier  #ZeroTier内网穿透



以下是全部：           注：应用后面标记 “ * ” 为最近新添加；标记“ ! ”与其他插件依赖或冲突。
-------------------------------------------------------------------------------------------------------------------
LuCI ---> Applications ---> luci-app-accesscontrol  #访问时间控制
LuCI ---> Applications ---> luci-app-acme  #ACME自动化证书管理环境（丢弃）
LuCI ---> Applications ---> luci-app-adblock   #ADB广告过滤
LuCI ---> Applications ---> luci-app-adbyby-plus  #广告屏蔽大师Plus +
LuCI ---> Applications ---> luci-app-adbyby   #广告过滤大师（丢弃）
LuCI ---> Applications ---> luci-app-adguardhome  #AdGuard home广告过滤（Le库以外的插件）
LuCI ---> Applications ---> luci-app-adkill   #广告过滤（丢弃）
LuCI ---> Applications ---> luci-app-advanced-reboot  #Linksys高级重启
LuCI ---> Applications ---> luci-app-advancedsetting  #系统高级设置（Le库以外的插件）
LuCI ---> Applications ---> luci-app-ahcp  #Ad-Hoc配置协议(AHCP) ipv6 and 双栈 自动配置协议 !
LuCI ---> Applications ---> luci-app-airplay2   #Apple AirPlay2 无损音频接收服务器
LuCI ---> Applications ---> luci-app-aliddns   #阿里DDNS客户端（丢弃，集成至ddns）
LuCI ---> Applications ---> luci-app-aliyundrive-webdav  #阿里云盘 WebDAV 服务
LuCI ---> Applications ---> luci-app-amule  #aMule下载工具 !
LuCI ---> Applications ---> luci-app-argon-config  #Argon主题配置插件（Le库以外的插件）
LuCI ---> Applications ---> luci-app-aria2 # Aria2下载工具
LuCI ---> Applications ---> luci-app-arpbind  #IP/MAC绑定
LuCI ---> Applications ---> luci-app-asterisk  #支持Asterisk电话服务器
LuCI ---> Applications ---> luci-app-attendedsysupgrade  #固件更新升级相关
LuCI ---> Applications ---> luci-app-autoreboot  #支持计划重启
LuCI ---> Applications ---> luci-app-baidupcs-web  #百度网盘管理
LuCI ---> Applications ---> luci-app-bcp38  #BCP38网络入口过滤（不确定）
LuCI ---> Applications ---> luci-app-bird1-ipv4  #对Bird1-ipv4的支持
LuCI ---> Applications ---> luci-app-bird1-ipv6  #对Bird1-ipv6的支持
LuCI ---> Applications ---> luci-app-bird4   #Bird 4（未知）（丢弃）
LuCI ---> Applications ---> luci-app-bird6   #Bird 6（未知）（丢弃）
LuCI ---> Applications ---> luci-app-bmx6  #BMX6路由协议
LuCI ---> Applications ---> luci-app-bmx7  #BMX7路由协议（丢弃）
LuCI ---> Applications ---> luci-app-caldav  #联系人（丢弃）
LuCI ---> Applications ---> luci-app-cifs-mount   #CIFS/SMB挂载设置
LuCI ---> Applications ---> luci-app-cifsd  #CIFS/SMB网络共享
LuCI ---> Applications ---> luci-app-cjdns  #加密IPV6网络相关
LuCI ---> Applications ---> luci-app-clamav  #ClamAV杀毒软件
LuCI ---> Applications ---> luci-app-clash  #Clash客户端（Le库以外的插件）
LuCI ---> Applications ---> luci-app-commands  #Shell命令模块
LuCI ---> Applications ---> luci-app-cshark  #CloudShark捕获工具
LuCI ---> Applications ---> luci-app-dawn  #分布式AP管理程序
LuCI ---> Applications ---> luci-app-ddns   #动态域名 DNS（集成阿里DDNS客户端）
LuCI ---> Applications ---> luci-app-diag-core   #core诊断工具
LuCI ---> Applications ---> luci-app-diskman   #磁盘管理工具
    luci-app-diskman ---> Include btrfs-progs   #新型的写时复制 (COW)
    luci-app-diskman ---> Include lsblk   #lsblk命令 用于列出所有可用块设备的信息
    luci-app-diskman ---> Include mdadm   #mdadm命令 用于创建、管理、监控RAID设备的工具
    luci-app-diskman ---> Include kmod-md-raid456   #RAID 4,5,6 驱动程序模块（丢弃）
    luci-app-diskman ---> Include kmod-md-linear   #RAID 驱动程序模块（丢弃）
LuCI ---> Applications ---> luci-app-dnscrypt-proxy  #DNSCrypt解决DNS污染
LuCI ---> Applications ---> luci-app-dnsfilter  #DNSFilter基于DNS的广告过滤
LuCI ---> Applications ---> luci-app-dnsforwarder  #DNSForwarder防DNS污染
LuCI ---> Applications ---> luci-app-dnspod  #DNSPod动态域名解析（丢弃）
LuCI ---> Applications ---> luci-app-docker  #Docker容器(dockerman更名为docker)
LuCI ---> Applications ---> luci-app-dump1090  #民航无线频率（不确定）
LuCI ---> Applications ---> luci-app-dynapoint  #DynaPoint（未知）
LuCI ---> Applications ---> luci-app-e2guardian   #Web内容过滤器
LuCI ---> Applications ---> luci-app-easymesh   #简单MESH(可有线+无线回程)
LuCI ---> Applications ---> luci-app-eqos  #基于IP地址限速（Le库以外的插件）
LuCI ---> Applications ---> luci-app-familycloud   #家庭云盘
LuCI ---> Applications ---> luci-app-fileassistant   #文件管理助手（Le库以外的插件）
LuCI ---> Applications ---> luci-app-filetransfer  #文件传输（可web安装ipk包）
LuCI ---> Applications ---> luci-app-firewall   #添加防火墙
LuCI ---> Applications ---> luci-app-flowoffload  #Turbo ACC网络加速（集成FLOW,BBR,NAT,DNS（丢弃，移至TurboACC）
LuCI ---> Applications ---> luci-app-freifunk-diagnostics   #freifunk组件 诊断（未知）（丢弃）
LuCI ---> Applications ---> luci-app-freifunk-policyrouting  #freifunk组件 策略路由（未知）（丢弃）
LuCI ---> Applications ---> luci-app-freifunk-widgets  #freifunk组件 索引（未知）（丢弃）
LuCI ---> Applications ---> luci-app-frpc   #内网穿透Frp客户端
LuCI ---> Applications ---> luci-app-frps   #内网穿透Frp服务端
LuCI ---> Applications ---> luci-app-fwknopd  #Firewall Knock Operator服务器
LuCI ---> Applications ---> luci-app-guest-wifi   #WiFi访客网络
LuCI ---> Applications ---> luci-app-gfwlist   #GFW域名列表（丢弃）
LuCI ---> Applications ---> luci-app-go-aliyundrive-webdav   #阿里云盘webdav协议(文件管理/同步等)  *
LuCI ---> Applications ---> luci-app-gost  #隐蔽的https代理（Le库以外的插件）
LuCI ---> Applications ---> luci-app-haproxy-tcp   #HAProxy负载均衡-TCP
LuCI ---> Applications ---> luci-app-hd-idle  #硬盘休眠
LuCI ---> Applications ---> luci-app-hnet  #Homenet Status家庭网络控制协议
LuCI ---> Applications ---> luci-app-https-dns-proxy  #通过HTTPS代理为DNS提供Web UI
LuCI ---> Applications ---> luci-app-ipsec-vpnd  #VPN服务器 IPSec
LuCI ---> Applications ---> luci-app-jd-dailybonus  #京东签到服务
LuCI ---> Applications ---> luci-app-kodexplorer  #KOD可道云私人网盘（与vnStat冲突 ! ）
LuCI ---> Applications ---> luci-app-kooldns  #VPN服务器 ddns替代方案（丢弃）
LuCI ---> Applications ---> luci-app-koolproxy  #KP去广告（丢弃）
LuCI ---> Applications ---> luci-app-lxc   #LXC容器管理
LuCI ---> Applications ---> luci-app-meshwizard #网络设置向导（丢弃）
LuCI ---> Applications ---> luci-app-minidlna   #完全兼容DLNA / UPnP-AV客户端的服务器软件
LuCI ---> Applications ---> luci-app-mjpg-streamer   #兼容Linux-UVC的摄像头程序
LuCI ---> Applications ---> luci-app-mtwifi  #MTWiFi驱动的支持 （丢弃）
LuCI ---> Applications ---> luci-app-mmc-over-gpio   #添加SD卡操作界面（丢弃）
LuCI ---> Applications ---> luci-app-multiwan   #多拨虚拟网卡（丢弃，移至syncdial）
LuCI ---> Applications ---> luci-app-mwan   #MWAN负载均衡（丢弃）
LuCI ---> Applications ---> luci-app-music-remote-center   #PCHiFi 数字转盘遥控
LuCI ---> Applications ---> luci-app-mwan3   #MWAN3负载均衡
LuCI ---> Applications ---> luci-app-mwan3helper   #MWAN3分流助手
LuCI ---> Applications ---> luci-app-n2n_v2   #N2N内网穿透 N2N v2 VPN服务
LuCI ---> Applications ---> luci-app-netdata  #Netdata实时监控（图形化）
LuCI ---> Applications ---> luci-app-nfs   #NFS网络共享
LuCI ---> Applications ---> luci-app-nft-qos  #QOS流控 Nftables版
LuCI ---> Applications ---> luci-app-ngrokc  #Ngrok 内网穿透（丢弃）
LuCI ---> Applications ---> luci-app-nlbwmon   #网络带宽监视器
LuCI ---> Applications ---> luci-app-noddos  #NodDOS Clients 阻止DDoS攻击
LuCI ---> Applications ---> luci-app-nps   #内网穿透nps
LuCI ---> Applications ---> luci-app-ntpc   #NTP时间同步服务器
LuCI ---> Applications ---> luci-app-ocserv  #OpenConnect VPN服务
LuCI ---> Applications ---> luci-app-olsr  #OLSR配置和状态模块
LuCI ---> Applications ---> luci-app-olsr-services  #OLSR服务器
LuCI ---> Applications ---> luci-app-olsr-viz   #OLSR可视化
LuCI ---> Applications ---> luci-app-ocserv   #OpenConnect VPN服务（丢弃）
LuCI ---> Applications ---> luci-app-openclash  #运行在OpenWrt上的Clash代理客户端（Le库以外的插件）
LuCI ---> Applications ---> luci-app-openvpn  #OpenVPN客户端
LuCI ---> Applications ---> luci-app-openvpn-server  #易于使用的OpenVPN服务器 Web-UI
LuCI ---> Applications ---> luci-app-oscam   #OSCAM服务器（丢弃）
LuCI ---> Applications ---> luci-app-p910nd   #打印服务器模块
LuCI ---> Applications ---> luci-app-pagekitec   #Pagekitec内网穿透客户端
LuCI ---> Applications ---> luci-app-passwall  #科学上网（Li大佬插件）
    Configuration ---> Include Brook  #Brook代理(跨平台强加密且不可检测代理)
    Configuration ---> Include ChinaDNS-NG  #防污染DNS服务
    Configuration ---> Include Haproxy  #HAProxy  #HAProxy负载均衡
    Configuration ---> Include Hysteria  #Hysteria双边加速工具
    Configuration ---> Include Kcptun  #Kcptun双边加速工具
    Configuration ---> Include NaiveProxy  #NaiveProxy代理(Chrome网络堆栈伪装流量)
    Configuration ---> Include PDNSD  #DNS服务器
    Configuration ---> Include Shadowsocks Libev Client  #SS Libev客户端(轻量级)
    Configuration ---> Include Shadowsocks Libev Server  #SS Libev服务端(轻量级)
    Configuration ---> Include Shadowsocks Rust Client  #SS Rust客户端(负载均衡/探测延迟)
    Configuration ---> Include ShadowsocksR Libev Client  #SSR Libev客户端(轻量级)
    Configuration ---> Include ShadowsocksR Libev Server  #SSR Libev服务端(轻量级)
    Configuration ---> Include Simple-Obfs (Shadowsocks plugin)  #simple-Obfs简单混淆工具(Nginx)
    Configuration ---> Include Trojan_GO  #Trojan_GO代理(直接模仿协议HTTPS)
    Configuration ---> Include Trojan_Plus  #Trojan_Plus代理(直接模仿协议HTTPS)
    Configuration ---> Include V2ray  #V2Ray代理
    Configuration ---> Include v2ray-plugin (Shadowsocks plugin)  #SS V2ray插件(WebSocket+TLS )
    Configuration ---> Include Xray  #Xray代理(XTLS)
    Configuration ---> Include Xray-Plugin (Shadowsocks Plugin)  #SS Xray插件(WebSocket+TLS )   *
    Configuration ---> Include Dns2socks  #DNS服务器（丢弃）
    Configuration ---> Include Redsocks2  #Redsocks2代理(透明TCP定向Socks/HTTPS代理服务器)（丢弃）
    Configuration ---> Include Shadowsocks  #SS代理（丢弃）
    Configuration ---> Include Shadowsocks Server  #SS服务器（丢弃）
    Configuration ---> Include Shadowsocks Rust (AEAD ciphers only)  #SS-RUST代理(AEAD加密)（丢弃）
    Configuration ---> Include ShadowsocksR   #SSR代理（丢弃）
    Configuration ---> Include ShadowsocksR Server  #SSR服务器（丢弃）
    Configuration ---> Include Https DNS Proxy(DoH)  #HttpsDNS服务（丢弃）
LuCI ---> Applications ---> luci-app-polipo  #Polipo代理(是一个小型且快速的网页缓存代理)
LuCI ---> Applications ---> luci-app-pppoe-relay  #PPPoE NAT穿透 点对点协议（PPP）
LuCI ---> Applications ---> luci-app-pptp-server  #VPN服务器 PPTP
LuCI ---> Applications ---> luci-app-privoxy  #Privoxy网络代理(带过滤无缓存)
LuCI ---> Applications ---> luci-app-ps3netsrv  #PS3 NET服务器(用于加载蓝光/游戏ISO/PKG)
LuCI ---> Applications ---> luci-app-pushbot  #全能推送(钉钉推送,企业微信推送,Bark,PushPlus推送)
LuCI ---> Applications ---> luci-app-qbittorrent  #BT下载工具(qBittorrent)
    Build Version Selection (Static Build)  ---> Static Build  #选择静态编译版本
    Build Version Selection (Static Build)  ---> Dynamic Build  #选择动态编译版本
LuCI ---> Applications ---> luci-app-qos   #流量服务质量(QoS)流控
LuCI ---> Applications ---> luci-app-radicale   #CalDAV/CardDAV同步工具
LuCI ---> Applications ---> luci-app-ramfree  #释放内存
LuCI ---> Applications ---> luci-app-rclone  #命令行云端同步工具
    Include rclone-webui  #Rclone界面
    Include rclone-ng (another webui)  #Rclone另一个界面
    Include fuse-utils (mount cloud storage)  #fuse-utils（挂载云存储）（丢弃）
LuCI ---> Applications ---> luci-app-rp-pppoe-server  #Roaring Penguin PPPoE Server 服务器
LuCI ---> Applications ---> luci-app-samba   #网络共享（Samba）
LuCI ---> Applications ---> luci-app-samba4   #网络共享（Samba4）
LuCI ---> Applications ---> luci-app-serverchan   #微信/Telegram推送的插件
LuCI ---> Applications ---> luci-app-sfe  #Turbo ACC网络加速（丢弃，移至TurboACC）
LuCI ---> Applications ---> luci-app-shadowsocks   #SS科学上网（丢弃）
LuCI ---> Applications ---> luci-app-shadowsocks-libes  #SS-libev服务端
LuCI ---> Applications ---> luci-app-shairplay  #支持AirPlay功能
LuCI ---> Applications ---> luci-app-siitwizard  #SIIT配置向导  SIIT-Wizzard
LuCI ---> Applications ---> luci-app-simple-adblock  #简单的广告拦截
LuCI ---> Applications ---> luci-app-smartdns  #SmartDNS本地服务器（丢弃）
LuCI ---> Applications ---> luci-app-softethervpn  #SoftEther VPN服务器  NAT穿透
LuCI ---> Applications ---> luci-app-splash  #Client-Splash是无线MESH网络的一个热点认证系统
LuCI ---> Applications ---> luci-app-sqm  #流量智能队列管理（QOS）
LuCI ---> Applications ---> luci-app-squid   #Squid代理服务器
LuCI ---> Applications ---> luci-app-ssr-plus   #SSR科学上网Plus+（Le大佬插件）
    Include libustream-ssl  ---> Include libustream-wolfssl  #选择wolfSSL库(传输层安全协议)
    Include libustream-ssl  ---> Include libustream-openssl  #选择OpenSSL库(传输层安全协议)
    luci-app-ssr-plus ---> Include Kcptun  #Kcptun双边加速工具
    luci-app-ssr-plus ---> Include NaiveProxy  #NaiveProxy代理(Chrome网络堆栈伪装流量)
    luci-app-ssr-plus ---> Include Redsocks2  #Redsocks2代理(透明TCP定向Socks/HTTPS代理服务器)
    luci-app-ssr-plus ---> Include Shadowsocks Libev Client  #SS Libev客户端(轻量级)
    luci-app-ssr-plus ---> Include Shadowsocks Libev Server  #SS Libev服务端(轻量级)
    luci-app-ssr-plus ---> Include Shadowsocks Rust Client  #SS Rust客户端(负载均衡/探测延迟)
    luci-app-ssr-plus ---> Include Shadowsocks Rust Server  #SS Rust服务端(负载均衡/探测延迟)
    luci-app-ssr-plus ---> Include ShadowsocksR Libev Client  #SSR Libev客户端(轻量级)
    luci-app-ssr-plus ---> Include ShadowsocksR Libev Server  #SSR Libev服务端(轻量级)
    luci-app-ssr-plus ---> Include Simple-Obfs Plugin  #SS Simple-Obfs混淆代理(Nginx)
    luci-app-ssr-plus ---> Include Trojan  #Trojan代理(直接模仿协议HTTPS)
    luci-app-ssr-plus ---> Include Shadowsocks V2ray Plugin  #SS V2ray代理(WebSocket+TLS )
    luci-app-ssr-plus ---> Include Xray  #Xray代理(XTLS)
    luci-app-ssr-plus ---> Include Shadowsocks New Version  #新SS代理（丢弃）
    luci-app-ssr-plus ---> Include Shadowsocks  #SS代理（丢弃）
    luci-app-ssr-plus ---> Include Shadowsocks Rust (AEAD ciphers only)  #SS-RUST代理(AEAD密码)  （丢弃）
    luci-app-ssr-plus ---> Include V2ray  #V2Ray代理（丢弃）
    luci-app-ssr-plus ---> Include Xray (V2RAY/Trojan-GO implemented)  #Xray代理（丢弃）
    luci-app-ssr-plus ---> Include Trojan-go  #Trojan-go代理（丢弃）
    luci-app-ssr-plus ---> Include Shadowsocks Server  #SS服务器（丢弃）
    luci-app-ssr-plus ---> Include Shadowsocks Rust Server  #SS Rust服务器（丢弃）
    luci-app-ssr-plus ---> Include ShadowsocksR Server  #SSR服务器（丢弃）
    luci-app-ssr-plus ---> Include DNS2SOCKS  #DNS服务器（丢弃）
    luci-app-ssr-plus ---> Include ShadowsocksR Socks and Tunnel（丢弃）
    luci-app-ssr-plus ---> Include Socks Server  #socks代理服务器（丢弃）
LuCI ---> Applications ---> luci-app-ssr-pro  #SSR-Pro（丢弃）
LuCI ---> Applications ---> luci-app-ssrserver-python  #ShadowsocksR Python服务器
LuCI ---> Applications ---> luci-app-statistics  #流量监控工具
LuCI ---> Applications ---> luci-app-syncdial  #多拨虚拟网卡（原macvlan）
LuCI ---> Applications ---> luci-app-tinyproxy  #Tinyproxy是 HTTP(S)代理服务器
LuCI ---> Applications ---> luci-app-transmission   #BT下载工具
LuCI ---> Applications ---> luci-app-travelmate  #旅行路由器
LuCI ---> Applications ---> luci-app-ttyd   #网页终端命令行
LuCI ---> Applications ---> luci-app-turboacc   #Turbo ACC 网络加速(支持 Fast Path 或者 硬件 NAT)
    luci-app-turboacc ---> Include Shortcut-FE  #Shortcut-FE 流量分载
    luci-app-turboacc ---> Include BBR CCA  #BBR拥塞控制算法提升TCP网络性能
    luci-app-turboacc ---> Include DNSForwarder  #DNS防污染 Forwarder
    luci-app-turboacc ---> Include DNSProxy  #DNS防污染 Proxy
LuCI ---> Applications ---> luci-app-udpxy  #udpxy做组播服务器
LuCI ---> Applications ---> luci-app-uhttpd  #uHTTPd Web服务器
LuCI ---> Applications ---> luci-app-unblockmusic  #解锁网易云灰色歌曲3合1新版本
    UnblockNeteaseMusic Golang Version  #Golang版本
    UnblockNeteaseMusic NodeJS Version  #NodeJS版本
LuCI ---> Applications ---> luci-app-unblockneteasemusic-go  #解除网易云音乐（合并）
LuCI ---> Applications ---> luci-app-unblockneteasemusic-mini  #解除网易云音乐（合并）
LuCI ---> Applications ---> luci-app-unbound  #Unbound DNS解析器
LuCI ---> Applications ---> luci-app-upnp   #通用即插即用UPnP（端口自动转发）
LuCI ---> Applications ---> luci-app-usb-printer  #USB 打印服务器
LuCI ---> Applications ---> luci-app-uugamebooster  #UU网游加速器
LuCI ---> Applications ---> luci-app-v2ray-server   #V2Ray 服务器
LuCI ---> Applications ---> luci-app-v2ray-pro  #V2Ray透明代理（丢弃，集成SSR）
LuCI ---> Applications ---> luci-app-verysync  #微力同步
LuCI ---> Applications ---> luci-app-vlmcsd  #KMS服务器设置
LuCI ---> Applications ---> luci-app-vnstat   #vnStat网络监控（图表）（与kodexplorer冲突 ! ）
LuCI ---> Applications ---> luci-app-vpnbypass  #VPN BypassWebUI  绕过VPN设置
LuCI ---> Applications ---> luci-app-vsftpd  #FTP服务器
LuCI ---> Applications ---> luci-app-vssr  #VSSR科学上网（je大佬插件）
    luci-app-vssr ---> Include Xray  #Xray代理(XTLS)
    luci-app-vssr ---> Include Trojan  #Trojan代理(直接模仿协议HTTPS)
    luci-app-vssr ---> Include Kcptun  #Kcptun双边加速工具
    luci-app-vssr ---> Include Shadowsocks Xray Plugin  #SS Xray代理
    luci-app-vssr ---> Include ShadowsocksR Libev Server  #SSR Libev服务端(轻量级)
LuCI ---> Applications ---> luci-app-watchcat  #断网检测功能与定时重启
LuCI ---> Applications ---> luci-app-webadmin  #Web管理页面设置
LuCI ---> Applications ---> luci-app-webshell  #网页命令行终端（丢弃）
LuCI ---> Applications ---> luci-app-wifischedule  #WiFi 计划
LuCI ---> Applications ---> luci-app-wireguard  #VPN服务器 WireGuard状态
LuCI ---> Applications ---> luci-app-wireless-regdb  #WiFi无线
LuCI ---> Applications ---> luci-app-wol   #WOL网络唤醒
LuCI ---> Applications ---> luci-app-wrtbwmon  #实时流量监测
LuCI ---> Applications ---> luci-app-xlnetacc  #迅雷快鸟
LuCI ---> Applications ---> luci-app-zerotier  #ZeroTier内网穿透
----------------------------------------------------------------------------------------
```