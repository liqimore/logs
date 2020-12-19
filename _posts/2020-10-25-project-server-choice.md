---
layout: post
title: "项目服务器选择，Linode vs Vultr vs Upcloud"
description: "upcloud对比linde,vultr等"
date: 2020-10-25
tags: [vps, upcloud评测, linode对比]
categories: [vps, 评测]
comments: true
---
# 0. intro

最近新的项目需要一台VPS，要求域名不需要备案，带宽以及流量大，国内访问速度尚可，稳定。经过考虑，首先排除了国内的商家（阿里云，腾讯云境外机房贵而且带宽小），其次排除国内外小商家（oneman），以免随时跑路。最终还是在Linode, Vultr 和 Upcloud等几个商家中。

除了上面提到的几个外，可选的还有例如Amazon Lightsail，GCE，DO，Krypt等，但是这几个都不符合我的要求，要么线路太差，要么价格太贵。

我需要缓存一定量的数据到内存中，内存大小也是我一个重要的考虑因素，虽然如此，但这几家的内存大小都和价格一致，也没什么办法了。在寻找VPS的过程中，我发现了一家德国公司Contabo，他们公司新开了一个美国机房，虽然线路不好，但是套个Cloudflare也不是不可用。Contabo可以做到13 usd给16G内存，400G的SSD，6个vCore（部分还是EYPC）以及400Mb的不限量带宽，非常诱人。

德国的厂商只用过Netcup的VDS，给我的印象是他们对客户的身份审核非常认真，Netcup我发送了信用卡以及护照+驾照的翻译件才通过审核（国际驾照翻译即可），所以对德国商家还是印象不错的。不过Contabo我需要的这款需要5 EUR的设置费，十分难受，就没有测试，如果之后10 usd的2G内存不够用，再考虑迁移吧。

# 1. Linode网络情况和跑分

Linode是一个非常好的选择，新用户需要信用卡激活，一下就可以筛选掉一大批不合规的用户，然后他们的带宽给的口子十分大，还可以挂在存储。为了测试，我选择了Tokyo 2的机房，高峰虽然会炸，但是大部分还是可以的，尤其是联通用户。规格是1C2G的。

![image.png](/assets/images/202010/973376286.png)

稍等片刻，VPS会完成启动。

![image.png](/assets/images/202010/2056853723.png)

山西联通PING（日常应该是70-80ms）。

![image.png](/assets/images/202010/217994619.png)

SSH登入系统，直接运行Superbench脚本。
`wget -qO- --no-check-certificate https://raw.githubusercontent.com/oooldking/script/master/superbench.sh | bash `

硬盘IO非常高，跑数据库也是非常可以了（如果做机场，上海联通中转，DNS解锁是个非常好的选择）。

![image.png](/assets/images/202010/587405460.png)

最后，我会跑一下秋水宜冰的Unixbench脚本，大致测试一下CPU性能。

![image.png](/assets/images/202010/531053427.png)

这个得分其实是非常高的，非常满意。

# 2. Vultr网络和跑分

Vultr和Linode类似，但是账户验证不是很严格，被许多人拿来翻墙，日本的IP经常开出不可用的。下面我同样选择日本机房以及1C2G的规格。

![image.png](/assets/images/202010/3924169500.png)

山西联通PING。Superbench的脚本获取IP的时候出错了，将就看一下。Vultr的IO还是差很多，只有高性能实例是NVME硬盘，不过网络情况很好，尤其是上海联通。。

![image.png](/assets/images/202010/2034025435.png)

最后是Unixbench跑分。

![image.png](/assets/images/202010/4263258077.png)

得分属于这个级别VPS正常的水平。

# 3. Upcloud网络和跑分

Upcloud是一个新商家，总部在欧洲，前一段时间新用户注册送25 usd，推荐另一个新用户直接送50 usd，所以我搞到了不少余额。Upcloud主要提供高性能的服务器，和Vultr的高性能实例类似，下面是它的基础测试以及Unixbench跑分。

Upcloud没有日本机房，亚洲只有新加坡，但是绕美国，所以我选了圣何塞机房测试。

![image.png](/assets/images/202010/977916428.png)

从山西联通的PING值。

![image.png](/assets/images/202010/1471542389.png)

测试脚本。

![image.png](/assets/images/202010/1912393677.png)

最后Unixbench。

![image.png](/assets/images/202010/523552476.png)

果然宣传性能的得分最高。

# 4. 总结

通过测试，可以看到Upcloud硬盘是被限制了IO的，本身性能应该更好，带宽是对等G口，另外两家都超过G口。同时，Vultr的CPU不是AMD，其他两家都是，其实我还是很喜欢EYPC。Vultr的高性能用的是E3，其实也没有很强。Vultr网络状况最好。

性能： Unixbench可以看出，Upcloud最好，毕竟性能作为卖点。

最后为了方便以及硬盘IO，我还是决定Linode。如果之后测试过Contabo，发现它更好或者内存16G完全可用，再做决定。
