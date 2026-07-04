# 搬瓦工 OVZ 切换 KVM 能直接转吗？真实答案 + 数据迁移完整教程：买哪个套餐、怎么搬数据、搬完能获得哪些提升

搬瓦工原来有一批卖得很火的 OpenVZ（OVZ）套餐，当时便宜，很多人冲着 $3.99、$4.99 的年付就买了。后来搬瓦工推了 KVM，然后你就开始好奇一件事：**搬瓦工 OVZ 切换 KVM，直接在后台能操作吗？**

答案直接说：**不行。**

OVZ 和 KVM 是两种根本不同的虚拟化架构，底层完全不兼容，搬瓦工官方也明确表示无法直接转换。官方客服的原话是：*"The only way to switch between OpenVZ and KVM is to order a new VPS and manually transfer all data."* 翻过来就是——唯一的办法是买台新的 KVM 机器，然后自己把数据搬过去，IP 也会跟着变。

快照迁移也走不通。有人试过直接用 KiwiVM 后台的 Snapshot 功能把 OVZ 快照导入 KVM，结果报错：`Import between KVM and OVZ technology is not possible`。死路一条。

---

## 为什么搬瓦工要淘汰 OVZ

这事不是突然的。OpenVZ 本质上是一种容器技术，所有 VPS 共享宿主机内核，你没有独立内核，也没法自己换内核——这意味着一件很实际的事：**装不了 BBR**。

BBR 是 Google 开源的 TCP 拥塞控制算法，对改善跨太平洋线路的实际体验提升明显，尤其是晚高峰。OVZ 的母机内核版本不够新，你自己没法改，只能干等。

KVM 是完全虚拟化，每个 VPS 有自己的独立内核，随便折腾。装 BBR、换内核、上 KVM 核显，都可以。超售比例也比 OVZ 低，实际稳定性更好。

搬瓦工大概在 2018 年底宣布所有 OVZ 套餐全面下线、已购的不能续费，然后延长了两周给大家搬数据。到现在，官网在售的全部套餐都是基于 KVM 架构。

---

## OVZ 切换 KVM 的正确路径：分三步

### 第一步：在 KVM 套餐里选一个新机器

这是绕不开的。不用急着删老机器，先把新的买好、跑起来。

OVZ 下线这么久，搬瓦工现在在售的套餐分几个档：

| 套餐系列 | 内存 | 硬盘 | 流量/月 | 带宽 | 可用机房 | 价格 |
|------|------|------|---------|------|---------|------|
| KVM 入门款 | 1 GB | 20 GB SSD | 1 TB | 1 Gbps | DC8 ZNET、DC2 AO、弗里蒙特、新泽西、纽约、加拿大、荷兰等 |  [$49.99/年](https://bandwagonhost.com/aff.php?aff=80238&pid=44) |
| KVM 中配款 | 2 GB | 40 GB SSD | 2 TB | 1 Gbps | 同上 |  [$99.99/年](https://bandwagonhost.com/aff.php?aff=80238&pid=45) |
| KVM 大配款 | 4 GB | 80 GB SSD | 3 TB | 1 Gbps | 同上 |  [$199.99/年](https://bandwagonhost.com/aff.php?aff=80238&pid=46) |
| CN2 GIA-E 入门 | 1 GB | 20 GB SSD | 1 TB | 2.5 Gbps | DC6 CN2 GIA-E、DC9 CN2 GIA、日本大阪软银、荷兰联通、圣何塞、迪拜等共 14 个 |  [$169.99/年](https://bandwagonhost.com/aff.php?aff=80238&pid=57) |
| CN2 GIA-E 中配 | 2 GB | 40 GB SSD | 2 TB | 2.5 Gbps | 同上 |  [$299.99/年](https://bandwagonhost.com/aff.php?aff=80238&pid=58) |
| CN2 GIA-E 大配 | 4 GB | 80 GB SSD | 3 TB | 2.5 Gbps | 同上 |  [$549.99/年](https://bandwagonhost.com/aff.php?aff=80238&pid=59) |

说句实话：如果你只是用来跑个科学上网或者轻量建站，KVM 入门款 $49.99/年完全够用，算下来一个月不到 5 美元。如果对速度有要求、在意晚高峰体验，直接上 CN2 GIA-E，可以在 14 个机房里切，DC6 和日本大阪软银是速度口碑最好的两个节点。

👉 [查看搬瓦工全部在售套餐对比](https://bit.ly/BanWaGon)

---

### 第二步：把 OVZ 上的数据打包迁移过来

买好新机器、SSH 能登上之后，就可以开始搬数据了。

**方法一：rsync 直接传（推荐）**

rsync 支持断点续传，传大文件不怕中途断。在 OVZ 机器上执行：

bash
rsync -avz --progress /你的数据目录/ root@新机器IP:/目标目录/


传之前先在新机器上确认 SSH 端口（搬瓦工默认不是 22，具体在 KiwiVM 后台的 SSH 信息里看）。如果端口不是 22，加个 `-e "ssh -p 端口号"` 参数：

bash
rsync -avz -e "ssh -p 你的端口" --progress /你的数据目录/ root@新机器IP:/目标目录/


**方法二：先打包再传**

如果文件比较分散，或者你想整个目录一起搬：

bash
# 在 OVZ 上打包
tar czf /tmp/backup.tar.gz /你的数据目录/

# 用 scp 或 rsync 传到新机器
rsync -avz -e "ssh -p 端口" /tmp/backup.tar.gz root@新机器IP:/tmp/

# 在新机器上解压
tar xzf /tmp/backup.tar.gz -C /目标目录/


传完之后，先别急着关 OVZ 机器，把服务在新机器上跑起来、验证正常了，再决定要不要删。

---

### 第三步：装 BBR，把 KVM 的优势用起来

搬过来之后做的第一件事应该是装 BBR。

在新的 KVM 机器上，KiwiVM 后台 → Reinstall OS → 选一个带 BBR 的系统镜像，直接装好，省得手动配置。或者自己跑一下开启脚本：

bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p


验证是否生效：

bash
sysctl net.ipv4.tcp_congestion_control
# 输出 bbr 就是成功了


OVZ 做不到这一步。KVM 上可以。这也是为什么同等线路条件下，KVM 的实际体验普遍比 OVZ 好一截。

---

## KVM 比 OVZ 好在哪，具体说

不想让你觉得这只是架构名字的差异。几个实际影响到你的点：

**可以自定义内核**。OVZ 用宿主机的内核，你动不了。KVM 上每台机器都有独立内核，内核太旧可以升，BBR 装得上，WireGuard 也装得上。

**稳定性更好**。OVZ 的母机超售比例通常更高，因为容器技术本身给了商家这个空间。KVM 是完全虚拟化，资源隔离更彻底，跑起来更稳。

**可以迁移到 CN2 机房**。KVM 套餐（常规款，非限量版）可以在 KiwiVM 后台一键切换到 CN2 机房，不用重新买机器。从普通 KVM 切到 CN2 后流量变为原来的三分之一，但线路质量提升明显，尤其电信用户体感差很多。OVZ 没有这个选项。

---

## 搬瓦工 KiwiVM 后台，迁移相关的功能在哪

买好 KVM 套餐之后，登录搬瓦工后台 → Services → My Services → 找到你的 VPS → 点 KiwiVM Control Panel。

几个你会用到的功能位置：

- **SSH 端口和 IP**：首页就能看到，记下来给 rsync 用
- **重装系统**：Reinstall OS，可以选带 BBR 的镜像
- **切换机房**：Migrate to another DC，普通 KVM 套餐可用
- **快照备份**：Snapshots，迁移前先做一个保底
- **流量使用情况**：首页右侧，传大文件前确认剩余流量够不够

---

## 常见问题

**Q：我的 OVZ 套餐已经过期了，还能拿到数据吗？**

过期之后系统通常保留几天，能不能开机要看具体情况。如果机器已经被删，数据基本是找不回来的。建议过期前及时提交工单问一下。

**Q：迁移之后 IP 会变吗？**

会。新买 KVM 机器，IP 是重新分配的，和 OVZ 的 IP 没有关系。如果域名解析指向了旧 IP，需要更新 DNS 记录。

**Q：OVZ 和 KVM 之间可以用快照互相迁移吗？**

不行。搬瓦工 KiwiVM 明确不支持跨架构快照导入，会直接报错。只能手动打包文件用 rsync/scp 传。

**Q：从 OVZ 迁到 KVM，现有的程序需要重新配置吗？**

新机器是全新环境，之前装的软件、配置文件都不会自动跟过来。要自己重装运行环境（比如 Nginx、数据库等），然后把数据文件传过来恢复。如果用 Docker 部署的，镜像和 volume 可以导出导入，相对方便一些。

**Q：KVM 套餐选哪个好？**

轻量用途（科学上网、跑个小脚本）：KVM 入门款 $49.99/年够了。建站或者有速度要求：上 CN2 GIA-E，季付 $49.99，可以在 14 个机房之间随意切，DC6 和日本软银节点在国内晚高峰稳得住。

👉 [立即选购搬瓦工 KVM 套餐，告别 OVZ 限制](https://bit.ly/BanWaGon)

---

## 最后说几句

搬瓦工 OVZ 切换 KVM 这件事，核心逻辑就一句话：**不能直接切，但迁过去之后值得**。

我自己把几台 OVZ 的东西陆续搬完之后，最直观的感受是装上 BBR 之后晚上的体验明显顺畅了，另外 CN2 GIA-E 的机房切换确实好用，哪个节点速度慢了换一个就行，不用重新买机器。

数据迁移的过程比想象中简单，rsync 跑一遍、环境重装一下，半天能搞定，前提是你知道自己机器上装了什么。趁着老机器还能用，早点搬，比到期了再抢时间强多了。

👉 [查看搬瓦工最新 KVM 套餐价格与机房配置](https://bit.ly/BanWaGon)
