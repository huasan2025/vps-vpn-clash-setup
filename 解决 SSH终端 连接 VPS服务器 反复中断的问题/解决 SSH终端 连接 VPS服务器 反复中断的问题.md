# 解决 SSH 终端连接 VPS 服务器反复中断的问题

SSH 登录 VPS：

```bash
ssh -i ~/.ssh/id_rsa.pem root@你的服务器IP
```

### 1. 先装 screen，防止再掉线

在服务器里执行：

```bash
apt update && apt install -y screen wget curl sudo
```

**常见问题：命令卡住不动，一直读秒**

正常情况几秒钟就能完成安装。如果长时间无响应，通常是 `apt` 锁文件被其他进程占用。

![卡住读秒的截图](images/解决%20SSH终端%20连接%20VPS服务器%20反复中断的问题-image.png)

报错信息通常长这样：

```
E: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 39362 (unattended-upgr)
```

注意其中的进程号（这里是 `39362`），每次都不同，需要根据你自己的报错信息替换。

#### 解决步骤

#### 1）先终止当前卡住的命令

按 `Ctrl + C` 终止当前的 `apt update && apt install` 命令。

#### 2）找到并杀死占用锁的进程

把下面命令中的 `进程号` 替换为报错信息里显示的实际数字：

```bash
# 先确认这个进程是什么
ps aux | grep 进程号

# 然后杀死它
kill 进程号
```

```bash
# 如果杀不掉，强制终止
kill -9 进程号
```

#### 3）手动删除锁文件

```bash
rm -f /var/lib/dpkg/lock-frontend
rm -f /var/cache/apt/archives/lock
rm -f /var/lib/dpkg/lock
```

#### 4）修复可能损坏的包状态 ⭐

```bash
dpkg --configure -a
```

#### 5）重新安装 screen

```bash
apt update && apt install -y screen wget curl sudo
```

这次应该就能秒装完成了 ✅

---

### 2. 开一个 screen 会话

```bash
screen -S v2
```

### 3. 重新执行安装

```bash
wget -P /root -N --no-check-certificate "https://raw.githubusercontent.com/mack-a/v2ray-agent/master/install.sh" && chmod 700 /root/install.sh && /root/install.sh
```

### 4. 如果又看到 sudoers 提示

输入：

```
N
```
