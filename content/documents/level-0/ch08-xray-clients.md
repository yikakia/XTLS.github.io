---
alwaysopen: false
date: "2021-01-05T00:00:00.000Z"
description: 小小白白话文
# head: <hr/>
hide:
# - toc
# - nextpage
post: "&nbsp;📙"
title: 【第8章】Xray客户端篇
weight: 8
---


## 8.1 Xray的工作原理简述

要正确的配置和使用`Xray`，就需要正确的理解其工作原理，对于新人，可以先看看下面的图（省略了许多复杂的设置）：

<img src="../ch08-img01-flow.png"  alt="Xray数据流向"/>

这其中的关键点是：

1. APP要主动或借助转发工具，将数据【流入(`inbounds`)】`Xray` 客户端

2. 流量进入客户端后，会被【客户端路由(`routing`)】按规则处理后，向不同方向【流出`(outbounds)`】`Xray` 客户端。比如：

    1. 国内流量直连（`direct`）
    2. 国外流量转发VPS（`proxy`）
    3. 广告流量屏蔽（`block`）

3. 向VPS转发的国外流量，会跨过防火墙，【流入(`inbounds`)】 `Xray` 服务器端

4. 流量进入服务器端后，与客户端一样，会被【服务器端路由(`routing`)】按规则处理后，向不同方向【流出`(outbounds)`】：
    1. 因为已经在防火墙之外，所以流量默认直连，你就可以访问到不存在网站们了（`direct`）
    2. 如果需要在不同的VPS之间做链式转发，就可以继续配置转发规则（`proxy`）
    3. 你可以在服务器端继续禁用各种你想禁用的流量，如广告、BT下载等（`block`）

{{% notice warning  %}}
**注意：** 请务必记得，`Xray` 的路由配置非常灵活，上面的说明只是无限可能性中的一种。

借助 `geosite.dat` 和 `geoip.dat` 这两个文件，可以很灵活的从【域名】和【IP】这两个角度、不留死角的控制流量流出的方向。这比曾经单一笼统的 `GFWList` 强大很多很多，可以做到非常细致的微调：比如可以指定Apple域名直连或转发、指定亚马逊域名代理或转发，百度的域名屏蔽等等。。。）
{{% /notice %}}


</br>

## 8.2 客户端与服务器端正确连接

现在你已经理解了 `Xray` 的工作原理，那么接下来的配置，其实就是【告诉你的客户端如何连接VPS服务器】。这和你已经很熟悉的、告诉`PuTTY`如何远程连接服务器是一样的。只不过Xray连接时的要素不止是【IP地址】+【端口】+【用户名】+【密码】这四要素了。

实际上，`Xray`的连接要素是由不同的[协议](../../../config/inbound-protocols/)决定的。本文在第7章的配置文件 `config.json` 里，我们使用 `Xray` 下独特而强大的 `VLESS` 协议 + `XTLS` 流控。所以那个配置文件就能知道，这个协议组合的连接要素有：

- 服务器【地址】: `a-name.yourdomain.com`
- 服务器【端口】: `443`
- 连接的【协议】: `vless`
- 连接的【流控】: `xtls-rprx-direct` (direct模式适合全平台，若是Linux/安卓用户，可改成 `xtls-rprx-splice` 性能全开)
- 连接的【验证】: `uuiduuid-uuid-uuid-uuiduuiduuid`
- 连接的【安全】: `"allowInsecure": false`

鉴于新人一般都会使用手机APP、或者PC端的客户端，种类繁多我就不再截图。我就把常用的客户端在后面罗列一下。但每个客户端都有自己的配置逻辑，所以，你要去仔细阅读这些客户端的说明、并把这些要素填入合适的地方。

-  **v2rayN - 适用于Windows平台**
    - 请从它的[GitHub仓库Release页面](https://github.com/2dust/v2rayN/releases)获取最新版
    - 请根据该客户端的说明进行设置

- **v2rayNG - 适用于Android平台**
    - 请从它的[GitHub仓库Release页面](https://github.com/2dust/v2rayNG/releases)获取最新版
    - 请根据该客户端的说明进行设置

- **Shadowrocket - 适用于iOS, 基于苹果M芯片的macOS**
    - 你需要注册一个【非中国区】的iCloud账户
    - 你需要通过 App Store 搜索并购买
    - 请根据该客户端的说明进行设置

- **Qv2ray - 跨平台图形界面，适用于Linux, Windows, macOS**
    - 请从它的[GitHub仓库Release页面](https://github.com/Qv2ray/Qv2ray/releases)获取最新版（还可以从它的[GitHub自动构建仓库](https://github.com/Qv2ray/Qv2ray/actions)寻找更新的版本）
    - 请从它的[项目主页](https://qv2ray.net/)学习文档
    - 请根据该客户端的说明进行设置


</br>

## 8.3 在PC端手工配置运行 `xray-core`

在PC端（Windows, macOS, Linux），我个人则比较推崇直接使用 `xray-core`，这样的优势是：

- 第一时间获得最新版而无需等待APP升级适配

- 灵活自由的路由配置能力（客户端中Qv2ray的高级路由编辑器非常强大，可以完整实现xray-core的路由功能）

- 节约一定的系统资源 （GUI界面一定会有资源消耗，消耗的多少则取决于客户端的实现）

唯一的劣势应该就是【需要手写配置文件】了。但其实，你想想，服务器上你已经成功的写过一次了，现在又有什么区别呢？接下来，还是老样子，我们分解一下步骤：

1. 首先请从Xray官方的 [GitHub仓库Release页面](https://github.com/XTLS/Xray-core/releases) 获取对应平台的版本，并解压缩到合适的文件夹
2. 在合适的文件夹建立空白配置文件：`config.json` （自己常用平台下新建文件大家肯定都会，这就真不用啰嗦了）
3. 至于什么是“合适的文件夹”？这就取决于具体的平台了~
4. 填写客户端配置
    - 我就以 `8.1` 原理说明里展示的基本三类分流（国内流量直连、国际流量转发VPS、广告流量屏蔽），结合 `8.2` 的连接要素，写成一个配置文件
    - 请将 `uuid` 替换成与你服务器一致的 `uuid`
    - 请将 `address` 替换成你的真实域名
    - 请将 `serverName` 替换成你的真实域名
    - 各个配置模块的说明我都已经（很啰嗦的）放在对应的配置点上了


    ```
    // REFERENCE:
    // https://github.com/XTLS/Xray-examples
    // https://xtls.github.io/config/
        
    // 常用的config文件，不论服务器端还是客户端，都有5个部分。外加小小白解读：
    // ┌─ 1_log          日志设置 - 日志写什么，写哪里（出错时有据可查）
    // ├─ 2_dns          DNS-设置 - DNS怎么查（防DNS污染、防偷窥、避免国内外站匹配到国外服务器等）
    // ├─ 3_routing      分流设置 - 流量怎么分类处理（是否过滤广告、是否国内外分流）
    // ├─ 4_inbounds     入站设置 - 什么流量可以流入Xray
    // └─ 5_outbounds    出站设置 - 流出Xray的流量往哪里去
        
        
    {
        // 1_日志设置
        // 注意，本例中我默认注释掉了日志文件，因为windows, macOS, Linux 需要写不同的路径，请自行配置
        "log": {
            // "access": "/home/local/xray_log/access.log",    // 访问记录
            // "error": "/home/local/xray_log/error.log"    // 错误记录
            "loglevel": "warning"        // 内容从少到多: "none", "error", "warning", "info", "debug" 
        },
        
        // 2_DNS设置
        "dns": {
            "servers": [
                // 2.1 国外域名使用国外DNS查询
                {
                    "address": "1.1.1.1",
                    "domains": [
                        "geosite:geolocation-!cn"
                    ]
                },
                // 2.2 国内域名使用国内DNS查询，并期待返回国内的IP，若不是国内IP则舍弃，用下一个查询
                {
                    "address": "233.5.5.5",
                    "domains": [
                        "geosite:cn"
                    ],
                    "expectIPs": [
                        "geoip:cn"
                    ]
                },
                // 2.3 作为2.2的备份，对国内网站进行二次查询
                {
                    "address": "114.114.114.114",
                    "domains": [
                        "geosite:cn"
                    ]
                },
                // 2.4 最后的备份，上面全部失败时，用本机DNS查询
                "localhost"
            ]
        },
        
        // 3_分流设置
        // 所谓分流，就是将符合否个条件的流量，用指定`tag`的出站协议去处理（对应配置的5.x内容）
        "routing": {
            "domainStrategy": "AsIs",
            "rules": [
                // 3.1 广告域名屏蔽
                {
                    "type": "field",
                    "domain": [
                        "geosite:category-ads-all"
                    ],
                    "outboundTag": "block"
                },
                // 3.2 国内域名直连
                {
                    "type": "field",
                    "domain": [
                        "geosite:cn"
                    ],
                    "outboundTag": "direct"
                },
                // 3.3 国内IP直连
                {
                    "type": "field",
                    "ip": [
                        "geoip:cn",
                        "geoip:private"
                    ],
                    "outboundTag": "direct"
                },
                // 3.4 国外域名代理
                {
                    "type": "field",
                    "domain": [
                        "geosite:geolocation-!cn"
                    ],
                    "outboundTag": "proxy"
                }
                // 3.5 默认规则
                // 在Xray中，任何不符合上述路由规则的流量，都会默认使用【第一个outbound（5.1）】的设置，所以一定要把转发VPS的outbound放第一个
            ]
        },

        // 4_入站设置
        "inbounds": [
            // 4.1 一般都默认使用socks5协议作本地转发
            {
                "tag": "socks-in",
                "protocol": "socks",
                "listen": "127.0.0.1",   // 这个是通过socks5协议做本地转发的地址
                "port": 10800,           // 这个是通过socks5协议做本地转发的端口
                "settings": {
                    "udp": true
                }
            },
            // 4.2 有少数APP不兼容socks协议，需要用http协议做转发，则可以用下面的端口
            {
                "tag": "http-in",
                "protocol": "http",
                "listen": "127.0.0.1",   // 这个是通过http协议做本地转发的地址
                "port": 10801            // 这个是通过http协议做本地转发的端口
            }
        ],
        
        // 5_出站设置
        "outbounds": [
        // 5.1 默认转发VPS
        // 一定放在第一个，在routing 3.5 里面已经说明了，这等于是默认规则，所有不符合任何规则的流量都走这个
            {
                "tag": "proxy",
                "protocol": "vless",
                "settings": {
                    "vnext": [
                        {
                            "address": "a-name.yourdomain.com",  // 替换成你的真实域名
                            "port": 443,
                            "users": [
                                {
                                    "id": "uuiduuid-uuid-uuid-uuid-uuiduuiduuid",  // 和服务器端的一致
                                    "flow": "xtls-rprx-direct",       // Windows, macOS 同学保持这个不变
                                    // "flow": "xtls-rprx-splice",    // Linux和安卓同学请改成Splice性能更强
                                    "encryption": "none",
                                    "level": 0
                                }
                            ]
                        }
                    ]
                },
                "streamSettings": {
                    "network": "tcp",
                    "security": "xtls",
                    "xtlsSettings": {
                        "serverName": "a-name.yourdomain.com",  // 替换成你的真实域名
                        "allowInsecure": false  // 禁止不安全证书
                    }
                }
            },
            // 5.2 用`freedom`协议直连出站，即当routing中指定'direct'流出时，调用这个协议做处理
            {
                "tag": "direct",
                "protocol": "freedom"
            },
            // 5.3 用`blackhole`协议屏蔽流量，即当routing中指定'block'时，调用这个协议做处理
            {
                "tag": "block",
                "protocol": "blackhole"
            }
        ]    
    }
    ```


</br>

## 8.4 圆满完成！

我相信，有耐心看到这里的同学，都是兼具好奇心和行动力的学习派！我现在要郑重的恭喜你，因为到了这里，你已经完完整整从头到尾全部完成了【**从第一条命令开始，完成VPS服务器部署，并成功在客户端配置使用科学上网**】的全过程！

我相信，你现在一定对`Linux`不再恐惧，对`Xray`不再陌生了吧！

**至此，小小白白话文圆满结束！**  `⬛⬛⬛⬛⬛⬛⬛⬛` **`100%`**


</br>

## 8.5 TO INFINITY AND BEYOND!

**但现在你看到的，远远不是Xray的全貌。**

`Xray`是一个强大而灵活的网络工具集合，平台化的提供了很多种能力，可以像瑞士军刀一样，通过灵活的配置组合解决各种问题。而本文，仅仅蜻蜓点水的用了**最简单**、**最直观**的配置来做**基础演示**。

如果你觉得现在已经完全够用了，那就好好的享受它给你带来的信息自由。但如果你的好奇心依然不能停歇，那就去继续挖掘它无限的可能性吧！

需要更多信息，可以到这里寻找：
1. [xtls.github.io](https://xtls.github.io/) - 官方文档站
2. [官方Telegram群组](https://t.me/projectXray) - 活跃而友善的官方讨论社区

<img src="../ch08-img02-buzz.png"  alt="TO INFINITY AND BEYOND!"/>


{{% notice warning  %}}
**不算后记的后记：**

希望我陪你走过的这一段小小的旅程，可以成为你网络生活中的一份小小助力。

这篇文章里的工具和信息难免会一点点的陈旧过时，但你一定会逐渐成长为大佬。未来的某个时间，若你能偶尔想起这篇教程、想起我写下本文的初衷，那我衷心希望你能够薪火相传、把最新的知识分享给后来人，让这一份小小的助力在社区里坚定的传递下去。

这是个大雪封山乌云密布的世界，人们孤独的走在各自的路上试图寻找阳光，如果大家偶尔交汇时不能守望相助互相鼓励，那最终剩下的，恐怕只有【千山鸟飞绝 万径人踪灭】的凄凉了吧。
{{% /notice %}}