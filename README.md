# 用干净 IP 访问 Claude：VPS + VPN + Clash 完整配置教程

> ⚠️ **重要说明**
>
> 本教程的目标是获得一个**相对干净的海外 IP**，降低因 IP 污染被封号的概率。
> **它不能保证彻底解决 Claude 封号问题。** 封号原因是多方面的（账号行为、支付方式、地区限制等），IP 只是其中一个因素。
>
> 请在充分了解上述前提后，再决定是否投入时间和金钱进行配置。

## 适用人群

本教程适合：

- 怀疑因 IP 污染导致 Claude / ChatGPT 访问异常或封号
- 需要一个干净、稳定的海外 IP 用于 AI 工具访问
- **Mac 用户**（Windows 用户 SSH 工具略有不同，其余步骤一致）

**预计费用：** $5 / 月起（VPS）

---

## 目录

1. [整体思路](#整体思路)
2. [第一步：购买 VPS 服务器](#第一步购买-vps-服务器)
   - [VMiss 购买步骤 & 线路选择](#vmiss-购买步骤)
   - [先查 IP 质量](#-先查-ip-质量)
3. [第二步：用 SSH 连接 VPS 服务器](#第二步用-ssh-连接-vps-服务器)
   - [VMiss：密码登录（推荐）](#vmiss密码登录推荐)
   - [DMIT：PEM 私钥登录](#dmitpem-私钥登录)
4. [第三步：在 VPS 上安装 VPN 服务](#第三步在-vps-上安装-vpn-服务)
   - [安装基础环境](#1-安装基础环境)
   - [安装 v2ray-agent](#2-安装-v2ray-agent)
   - [选择协议](#3-选择协议)
   - [获取订阅链接](#4-获取订阅链接)
5. [第四步：安装 Clash 并导入订阅](#第四步安装-clash-并导入订阅)
   - [安装 Clash Verge Rev](#1-安装-clash-verge-rev)
   - [导入订阅链接](#2-导入订阅链接)
   - [网络设置（TUN + 规则模式）](#3-网络设置)
   - [DNS 定向优化（可选）](#4-dns-定向优化可选)
   - [浏览器防泄露（推荐）](#5-浏览器防泄露推荐)
6. [第五步：验证配置是否成功](#第五步验证配置是否成功)
   - [基础验证](#基础验证)
   - [进阶验证：IP 质量检测脚本](#进阶验证可选)
7. [常见问题排查](#常见问题排查)
   - [服务器突然断连（Ubuntu 自动更新）](#服务器突然断连--无法访问)

---

## 整体思路

1. 购买海外 **VPS** 云服务器（推荐 [VMiss](https://app.vmiss.com/aff.php?aff=4686)，自用）
2. 用 **SSH** 连接 VPS 服务器
3. 在 VPS 上搭建 **VPN 服务**（使用 [v2ray-agent](https://github.com/mack-a/v2ray-agent)）
4. 把生成的订阅链接导入本地 **Clash**，开启 TUN 模式

> 想了解这套方案的原理？→ [为什么要在 VPS 上配置 VPN 服务？](docs/why-vps-vpn.md)

---

## 第一步：购买 VPS 服务器

推荐两个相对靠谱的 VPS 服务商：

| 服务商 | 价格 | 备注 |
|--------|------|------|
| **[VMiss](https://app.vmiss.com/aff.php?aff=4686)** | $5 / 月起 | ✅ 自用，IP 质量好，线路选择多 |
| [DMIT](https://www.dmit.io/aff.php?aff=19146) | $9.9 / 月起 | 备选，稳定性好 |

> 💡 如果觉得本教程有帮助，购买时可以使用我的推广链接，对你没有任何额外费用。

### VMiss 购买步骤

注册 VMiss 无需手机号，直接用邮箱即可。

**选套餐前，先确认你的宽带运营商**，不同线路对不同运营商的优化差异很大：

| 线路 | 最优运营商 | 说明 |
|------|-----------|------|
| **CN2 GIA** | 电信 | 电信最强，三网偏向电信优化 |
| **9929** | 联通 | 联通最强 |
| **CMIN2** | 移动 | 移动最强 |
| **TRI / #DC2** | 三网均衡 | 最推荐新手或家庭多运营商场景 |

> 以上均为针对中国大陆优化的高端线路，远优于普通国际线路（163 / 4837 / CMI），晚高峰拥堵少、速度稳。

选择建议：

- 知道自己运营商 → 选对应最优线路
- 不确定 / 家里多个运营商 → 选 **TRI** 或 **#DC2**
- 先买 1 个月体验，IP 质量不好再换

购买完成后，在控制台可以看到服务器的 **IPv4 地址**和**登录密码**。

---

### ✅ 先查 IP 质量

买完先别急着配置，先检测这个 IP 是否干净。

**网页版快速检测**

访问 [meowvps.com/tools/ip-check](https://meowvps.com/tools/ip-check/)，输入服务器 IPv4 地址。

> 想了解 IPv4 与 IPv6 的区别？→ [IPv4 和 IPv6 的区别](docs/ipv4-vs-ipv6.md)

💡 如果 IP 质量差，可以联系客服申请换 IP，或在退款窗口内申请退款换其他服务商。

**更全面的检测** → 完成全部配置后，在第五步的「进阶验证」中可以用服务器端脚本检测 IP 风险评分、端口开放状态、流媒体解锁情况。

---

## 第二步：用 SSH 连接 VPS 服务器

> **SSH** 是一种安全的远程连接协议，让你在本地终端操作远程服务器。

### VMiss：密码登录（推荐）

VMiss 默认使用密码登录，步骤更简单：

1. 打开 Mac 上的「终端」（Terminal）
2. 输入 SSH 命令（把 `你的服务器IP` 替换为实际 IPv4 地址）：

   ```bash
   ssh root@你的服务器IP -p 22
   ```

3. 首次连接会出现确认提示，输入 `yes` 回车
4. 输入控制台里的登录密码（输入时终端不会显示星号，正常输入即可）

### DMIT：PEM 私钥登录

如果你使用 DMIT，操作略有不同：

1. 在 DMIT 控制台点击 **Download**，下载 PEM 私钥文件

   ![下载 PEM 文件](images/pem-download.png)

2. 把私钥文件移动到 SSH 目录：

   ```bash
   mv ~/Downloads/你下载的文件夹名/id_rsa.pem ~/.ssh/
   ```

3. **修改文件权限**（必做，否则 SSH 会报错）：

   ```bash
   chmod 600 ~/.ssh/id_rsa.pem
   ```

4. 连接服务器：

   ```bash
   ssh -i ~/.ssh/id_rsa.pem root@你的服务器IP
   ```

> **Windows 用户**：下载 `.ppk` 格式的私钥，使用 [MobaXterm](https://mobaxterm.mobatek.net/)（免费推荐）或 Xshell 连接。

---

## 第三步：在 VPS 上安装 VPN 服务

使用开源项目 [v2ray-agent](https://github.com/mack-a/v2ray-agent)，一键脚本，无需手动配置。

**推荐方案：Xray + VLESS Reality（无域名版）**

| | **VLESS Reality** ✅ 推荐 | Hysteria2 |
|--|--|--|
| **稳定性** | 最稳，最适合防封 | 很快，个别环境略逊 |
| **速度** | 快 | **极快** |
| **抗封锁** | 最强（伪装正常 HTTPS 流量） | 强，不如 Reality |
| **适合场景** | 日常访问、防封首选 | 看视频、下载大文件 |

### 1. 安装基础环境

SSH 登录 VPS 后执行：

```bash
apt update && apt install -y wget curl sudo
```

### 2. 安装 v2ray-agent

```bash
wget -P /root -N --no-check-certificate "https://raw.githubusercontent.com/mack-a/v2ray-agent/master/install.sh" && chmod 700 /root/install.sh && /root/install.sh
```

脚本启动后，遇到 sudoers 相关提示选 **`N`**。

![v2ray-agent 安装界面](images/v2ray-install.png)

### 3. 选择协议

安装完成后，在菜单里选择：**Xray 内核 → VLESS Reality（无域名版）**

详细操作参考 [一键无域名版 Reality 安装教程](https://www.v2ray-agent.com/archives/1708584312877)

> 🚨 **遇到 SSH 连接反复中断？** → [解决 SSH 终端连接 VPS 反复中断的问题](docs/fix-ssh-disconnect.md)

### 4. 获取订阅链接

安装完成后，执行以下命令：

```bash
vasma
```

依次选择 `7` → `2` → 一路回车，得到 **clashMeta 订阅链接**：

```
http://你的服务器IP:端口/s/clashMetaProfiles/xxxxxxxxxxxxxxxx
```

复制这个完整 URL，下一步要用。

---

## 第四步：安装 Clash 并导入订阅

### 1. 安装 Clash Verge Rev

下载地址：[Clash Verge Rev Releases](https://github.com/clash-verge-rev/clash-verge-rev/releases)

- Mac：下载 `.dmg` 文件
- Windows：下载 `.exe` 文件

安装并打开。

### 2. 导入订阅链接

1. 点击左侧「**订阅**」
2. 把复制好的订阅链接粘贴进去
3. 点击「**导入**」

![导入订阅成功](images/clash-import.png)

### 3. 网络设置

进入「**设置**」，配置以下两项：

1. **TUN 模式** → 开启（需要输入系统密码授权）
2. **代理模式** → 选择「**规则**」

> TUN 模式会接管**全部**系统流量，避免 IPv6 泄漏导致真实 IP 暴露给目标网站。
> 代理模式选「规则」，国内流量直连、海外流量走代理，兼顾速度和访问体验。

### 4. DNS 定向优化（可选）

如果你主要用于访问 Claude、ChatGPT、Gemini 等 AI 工具，可以在 Clash 配置中加入以下 DNS fallback 规则，让这些域名优先走海外 DNS 解析，减少解析漂移：

```yaml
fallback-filter:
  domain:
    - +.anthropic.com
    - +.claude.ai
    - +.openai.com
    - +.chatgpt.com
    - +.google.com
    - +.googleapis.com
    - +.gstatic.com
```

> 这一步只影响列出的域名，不改变国内站点的解析路径。如果访问 AI 工具已经正常，可以跳过。

### 5. 浏览器防泄露（推荐）

Clash 只能处理网络流量，处理不了浏览器主动泄露的信息。以下两项建议在验证之前完成。

**关闭 WebRTC（必做）**

WebRTC 是最常见的 IP 泄露源。即使开了代理，WebRTC 也能通过 UDP 协议直接暴露你的真实 IP。

- Chrome / Edge：安装扩展 **WebRTC Control** 并开启

  ![WebRTC Control 扩展](images/WebRTC%20Control.png)

- 验证是否生效：访问 [browserleaks.com/webrtc](https://browserleaks.com/webrtc)，确认看不到你的真实 IP

> 重度用户也可以使用指纹浏览器（如 AdsPower），普通用户装插件即可。

**关闭 QUIC / HTTP3（推荐）**

Chrome 默认开启 QUIC 协议（基于 UDP），部分场景下 Clash 对 UDP 的拦截不如 TCP 稳定，可能导致流量绕过代理直连。

- Chrome 地址栏输入：`chrome://flags/#enable-quic`
- 设置为 **Disabled**

  ![关闭 QUIC 协议](images/QUIC%20protocol.png)

---

## 第五步：验证配置是否成功

### 基础验证

1. 访问 [myip.com](https://myip.com)，确认显示的是你的 **VPS IP**，而非本地网络 IP
2. 访问 [claude.ai](https://claude.ai)，确认可以正常打开

两项都通过，配置完成 ✅

### 进阶验证（可选）

想要更全面地了解你的 IP 质量？

**1. SSH 登录服务器**

```bash
ssh root@你的服务器IP -p 22
```

**2. 执行 IP 质量检测**

```bash
bash <(curl -Ls IP.Check.Place)
```

检测内容包括：IP 风险评分、端口开放状态、流媒体解锁情况。

---

## 常见问题排查

### 服务器突然断连 / 无法访问

**现象**：VPS 运行一段时间后突然无法连接，重启后恢复，但过几天再次发生。

**原因**：Ubuntu 系统的**自动更新**在后台执行，重启系统后服务中断。

**解决方案**：关闭自动更新，改为手动维护。

**第一步：关闭自动更新**

SSH 登录服务器后执行：

```bash
systemctl disable --now apt-daily.timer
systemctl disable --now apt-daily-upgrade.timer
systemctl disable --now unattended-upgrades.service
```

确认已关闭：

```bash
systemctl list-timers | grep apt
systemctl status unattended-upgrades --no-pager
```

**第二步：定期手动更新**（建议每月一次）

```bash
apt update
apt upgrade -y
reboot
```

> 手动更新后服务器会重启，重启完成后 VPN 服务会自动恢复。

---

## 推广链接

如果本教程对你有帮助，购买时可以使用以下推广链接，对你没有任何额外费用：

- **VMiss（自用推荐）**：[app.vmiss.com/aff.php?aff=4686](https://app.vmiss.com/aff.php?aff=4686)
- **DMIT**：[dmit.io/aff.php?aff=19146](https://www.dmit.io/aff.php?aff=19146)

---

## 致谢

本教程基于以下开源项目，感谢作者的贡献：

- [v2ray-agent](https://github.com/mack-a/v2ray-agent) — VPN 服务端安装脚本
- [Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev) — 本地代理客户端
- [IPQuality](https://github.com/xykt/IPQuality) — IP 质量检测脚本

---

*遇到问题？欢迎在 [Issues](https://github.com/huasan2025/vps-vpn-clash-setup/issues) 里提问。*
