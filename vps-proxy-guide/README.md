# 3X-UI + Vultr 搭建 VPS 完全指南（小白版）

> 💡 本文假设你完全没用过 VPS。每一步都会告诉你在哪操作、为什么这么做。跟着走就行。

---

## 第一步：买个 VPS

**VPS = 一台在国外的小电脑。** 你在国内上不了 Google，因为墙挡住了。VPS 在墙外面，你先连它，它帮你去拿 Google 的内容，再传回给你——它就是个"替身"。

### 📍 在哪买？

打开浏览器，访问 **vultr.com**（全英文，用 Chrome 自带翻译即可）。

### 🛒 买的步骤

**1️⃣ 注册账号**

右上角点「Sign Up」→ 用邮箱注册 → 邮箱里激活。

**2️⃣ 充值**

登录后点左边「Billing」→「Make Payment」→ 选支付宝（Alipay）→ 充 $10（约 70 块）。

> ❓ 为什么充 $10？最便宜的 VPS 是 $6/月，$10 够用一个月。之后可以每月自动扣。

**3️⃣ 创建 VPS**

点左边「Products」→ 右上角蓝色「Deploy +」→ 选「Deploy New Server」，按下表选：

| 选项 | 选什么 | 为什么 |
|------|--------|--------|
| Choose Server | Cloud Compute - Regular | 最便宜的，够用了 |
| CPU & Storage | $6/月（1核1G） | 翻墙不需要高性能 |
| Server Location | Los Angeles 或 Tokyo | 离中国近，速度快 |
| Image | Ubuntu 24.04 x64 | 最常用的 Linux |
| 其他 | 全部默认 | 不需要改 |

点「Deploy Now」，等状态变成 **Running**。

**4️⃣ 记下 VPS 信息**

点进服务器详情页：

| 要找什么 | 在哪看 | 长什么样 |
|----------|--------|----------|
| IP 地址 | 页面中间大号字 | 类似 `1.2.3.4` |
| 用户名 | 左侧 Username | 固定是 `root` |
| 密码 | 左侧 Password | 点眼睛复制 |

> ✅ 三样记好，后面每步都用。

---

## 第二步：登录 VPS

### 📍 在哪操作？

你电脑的**终端**：
- **Mac**：按 `Cmd+空格` 搜「终端」
- **Windows**：按 `Win+R` 输入 `powershell`

> ❓ 终端就是你跟 VPS 打字聊天的窗口，VPS 没屏幕没鼠标。

### 🔑 登录命令（把 IP 换成你的）

```bash
ssh root@你的VPS-IP
```

第一次会问 `yes/no`，输 `yes` 回车。然后输密码。

> ⚠️ 输密码时屏幕**不会显示任何东西**，正常，直接回车。

看到 `root@vultr:~#` 就成功了。`root` 是最高权限用户，装软件改配置都需要。

> ✅ 终端出现 `root@vultr:~#` 代表登录成功。

---

## 第三步：初始化 VPS

### 📍 还在终端里，VPS 上。

**更新系统：**

```bash
apt update && apt upgrade -y
```

- `apt update`：刷新软件列表
- `apt upgrade -y`：升级到最新版

**装依赖工具：**

```bash
apt install -y curl sqlite3
```

- `curl`：用来下载文件，后面每步都用
- `sqlite3`：出问题时排查数据库用

等 1-2 分钟，提示符重新出现即可。

> ✅ 终端没报错，提示符重新出现。

---

## 第四步：安装 3X-UI

### 这是什么？

一个**网页管理面板**。不用它要手写 JSON 配置，有了它全在网页上点。

### 📍 还在终端，VPS 上。

```bash
bash <(curl -Ls https://raw.githubusercontent.com/MHSanaei/3x-ui/master/install.sh)
```

> 💡 这条命令从 GitHub 下载安装脚本自动跑，全程不用管。

安装过程会问你：

| 问你什么 | 填什么 | 说明 |
|----------|--------|------|
| 面板端口 | 自己定，比如 `54321` | 打开管理页面的"门牌号" |
| 面板用户名 | 自己定，比如 `admin` | 登录账号 |
| 面板密码 | 自己定 | 别太简单 |

装完会显示 **面板地址: `http://你的VPS-IP:你设的端口/`**

> ✅ 记下面板地址、账号、密码。

---

## 第五步：搞定 TLS 证书

### TLS = 加密。

不加 TLS，GFW 能直接看到你的数据内容和协议类型。加了就看不出。

### 📍 还在终端，VPS 上。

> 📢 有域名选方案 A，没域名选方案 B。域名在阿里云/腾讯云/Cloudflare 几十块一年。

### 方案 A：有域名

```bash
# 装证书申请工具
curl https://get.acme.sh | sh

# 申请证书（替换成你的域名）
~/.acme.sh/acme.sh --issue -d 你的域名.com --standalone
```

> ⚠️ 先把 Vultr Firewall 的 **80 端口**打开，acme.sh 需要它验证域名。

```bash
# 复制到指定位置
mkdir -p /root/cert
~/.acme.sh/acme.sh --install-cert -d 你的域名.com \
  --key-file /root/cert/privkey.pem \
  --fullchain-file /root/cert/fullchain.pem
```

### 方案 B：没域名（用 nip.io）

`nip.io` 是免费服务，`你的IP.nip.io` 自动变成域名：

```bash
mkdir -p /root/cert
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /root/cert/privkey.pem \
  -out /root/cert/fullchain.pem \
  -subj "/CN=你的VPS-IP.nip.io"
```

> 💡 自签证书浏览器会报警告，但不影响翻墙——翻墙软件走程序连接，不走浏览器。

> ✅ `/root/cert/` 下有 `privkey.pem` 和 `fullchain.pem` 两个文件。

---

## 第六步：打开面板，配翻墙服务

### 6.1 打开面板

先去 **Vultr Firewall** 开端口，否则浏览器连不上。添加两条规则：

- TCP 端口：**你设的面板端口**（如 54321），Source：`0.0.0.0/0`
- TCP 端口：**443**，Source：`0.0.0.0/0`

然后浏览器打开 `http://你的VPS-IP:你的面板端口/`，用第四步的账号密码登录。

### 6.2 添加入站规则

"入站规则"告诉 xray：有人通过 443 端口来找你时，用什么暗号和加密方式说话。

📍 面板左侧 → **入站列表** → **添加入站**。

| 设置项 | 填什么 | 说明 |
|--------|--------|------|
| 备注 | `我的节点` | 方便区分 |
| 协议 | `vmess` | 最稳定 |
| 端口 | `443` | HTTPS 标准端口 |
| 传输 | `ws`（WebSocket） | 伪装成网页流量 |
| TLS | 开启 | 加密 |
| 公钥文件 | `/root/cert/fullchain.pem` | 第五步的证书 |
| 密钥文件 | `/root/cert/privkey.pem` | 第五步的私钥 |
| 路径 | `/vmess` | 自己定 |

点**添加**。

### 6.3 添加客户端

入站规则右侧三个点 → **客户端** → **添加**。会自动生成 UUID，类似 `1f596a3b-5df5-4b69-912d-b996b7d7801f`。

> ✅ 记下 UUID，后面配置文件要用。

---

## 第七步：在国内服务器上装客户端

### 先搞清楚：

VPS 是**服务端**（接收请求），国内服务器是**客户端**（发送请求），两边都要装 xray。

📍 **新开终端**，SSH 登录**国内服务器**（不是 VPS）。

### 7.1 装 xray

```bash
bash <(curl -Ls https://raw.githubusercontent.com/MHSanaei/3x-ui/master/install.sh) --install-xray
```

> 💡 `--install-xray` 表示只装核心，不装管理面板。

### 7.2 写配置文件

```bash
nano /usr/local/etc/xray-client.json
```

完整粘贴以下内容（**替换三处标注**）：

```json
{
  "inbounds": [
    {
      "port": 10808,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": { "udp": true }
    },
    {
      "port": 10809,
      "listen": "127.0.0.1",
      "protocol": "http"
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "填你的VPS-IP",
            "port": 443,
            "users": [{ "id": "填你的UUID" }]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "tlsSettings": { "serverName": "填你的VPS-IP.nip.io" },
        "wsSettings": { "path": "/vmess" }
      }
    }
  ]
}
```

> 📝 需要改的三处：`address` → VPS IP，`users.id` → UUID，`serverName` → IP.nip.io

### 7.3 设开机自启

```bash
cat > /etc/systemd/system/xray-client.service << 'EOF'
[Unit]
Description=Xray Client
After=network.target

[Service]
ExecStart=/usr/local/bin/xray run -c /usr/local/etc/xray-client.json
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now xray-client
```

### 7.4 测试

```bash
curl --socks5-hostname 127.0.0.1:10808 https://www.google.com
```

> ⚠️ 关键！注意是 `socks5-hostname`，不是 `socks5`！DNS 解析在 VPS 做才能拿到真的 Google IP。

> ✅ 检查点：看到一大片 HTML 乱码就对了！

### 7.5 便捷开关

```bash
# 编辑 ~/.bashrc，加到最后两行
alias proxy-on='export http_proxy=http://127.0.0.1:10809 https_proxy=http://127.0.0.1:10809'
alias proxy-off='unset http_proxy https_proxy'

# 让别名生效
source ~/.bashrc
```

之后 `proxy-on` 开代理，`proxy-off` 关代理。

---

## 第八步（可选）：把面板藏起来

### 为什么要藏？

目前你的面板是 `http://VPS-IP:54321/`。如果有人在网上扫端口，扫到你的面板端口就知道你在用 3X-UI。

藏起来后，面板藏在 443 端口下面的一个秘密路径里——外面看到的就是普通 HTTPS 网站。

### 8.1 装 Caddy

```bash
apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | \
  gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | \
  tee /etc/apt/sources.list.d/caddy-stable.list
apt update
apt install -y caddy
```

### 8.2 把 VMess 退到后台

回到 3X-UI 面板页面 → 入站列表 → 点**编辑**：

| 改什么 | 改成什么 | 为什么 |
|--------|----------|--------|
| 端口 | `10001` | 把 443 让给 Caddy |
| 监听地址 | `127.0.0.1` | 只在本机内部听 |
| TLS | 关闭 | Caddy 会处理 TLS |

保存后执行 `systemctl restart x-ui`。

### 8.3 配置 Caddy

```bash
nano /etc/caddy/Caddyfile
```

写入：

```caddyfile
{
    auto_https off
    http_port 9999
}

你的VPS-IP.nip.io:443 {
    tls /etc/caddy/certs/fullchain.pem /etc/caddy/certs/privkey.pem

    handle /你的秘密路径* {
        reverse_proxy 127.0.0.1:你的面板端口
    }

    handle /vmess* {
        reverse_proxy 127.0.0.1:10001
    }

    handle { respond 404 }
}
```

### 8.4 复制证书 + 启动

```bash
mkdir -p /etc/caddy/certs
cp /root/cert/*.pem /etc/caddy/certs/
chmod 644 /etc/caddy/certs/fullchain.pem
chmod 600 /etc/caddy/certs/privkey.pem

mkdir -p /etc/systemd/system/caddy.service.d
cat > /etc/systemd/system/caddy.service.d/override.conf << 'EOF'
[Service]
User=root
Group=root
EOF

systemctl daemon-reload
systemctl enable --now caddy
```

### 8.5 验证

```bash
# 面板（应返回 200）
curl -sk https://你的VPS-IP.nip.io/你的秘密路径/

# 根路径（应返回 404，安全）
curl -sk https://你的VPS-IP.nip.io/

# VMess（应返回 400 Bad Request，正常）
curl -sk https://你的VPS-IP.nip.io/vmess
```

### 8.6 关掉面板端口

```bash
iptables -I INPUT -p tcp --dport 你的面板端口 -s 127.0.0.1 -j ACCEPT
iptables -A INPUT -p tcp --dport 你的面板端口 -j DROP
```

### 🏗️ 最终架构

```
你国内服务器 → [xray 客户端] → [墙] → [VPS Caddy :443]
                                            ├── /vmess → xray VMess
                                            └── /秘密路径 → 3X-UI 面板
```

| 你得到什么 | 在哪用 |
|------------|--------|
| 国内服务器翻墙 | `proxy-on` 开启代理 |
| 管理面板 | 浏览器打开 `https://VPS-IP.nip.io/秘密路径/` |
| 新增用户 | 面板里点客户端 → 添加 |
| 客户端开机自启 | 不用管，服务器重启后自动连接 |

---

## 🛠️ 真遇到问题了怎么办

### 第一：查日志

```bash
# VPS 服务端
systemctl status x-ui
journalctl -u x-ui -n 20

# 客户端
systemctl status xray-client
journalctl -u xray-client -n 20
```

### 第二：查端口

```bash
ss -tlnp | grep 443    # VPS 上谁在监听
ss -tlnp | grep 10808  # 客户端
```

### 第三：查数据库

```bash
sqlite3 /etc/x-ui/x-ui.db "SELECT username,password FROM users"
sqlite3 /etc/x-ui/x-ui.db "SELECT id,port,protocol,enable FROM inbounds"
sqlite3 /etc/x-ui/x-ui.db "SELECT * FROM clients"
```

### 常见错误速查

| 症状 | 可能原因 | 在哪查 |
|------|----------|--------|
| curl 连不上 | Firewall 没开 443 | Vultr Firewall 页面 |
| Google 打不开 | 用了 `--socks5` 而不是 `--socks5-hostname` | 改命令 |
| 客户端日志报错 | UUID 写错了 | 面板里核对 UUID |
| Caddy 启动失败 | 证书路径不对或权限问题 | `systemctl status caddy` |
| 面板 404 | 路径写错了 | 核对 Caddyfile 里的路径 |

---

## 🎉 恭喜！

从头到尾跟下来，你现在有一台能翻墙的 VPS 了。

---

## 附录：Coze的云端OpenClaw 无脑操作版

> 💡 如果你有 Coze的云端OpenClaw，上面所有步骤都不用自己动手。把下面这段话发给它，全自动搞定。

### 什么是 Coze的云端OpenClaw？

一个部署在 Coze 云上的 AI 助手，能在你服务器上直接执行命令、装软件、写配置——相当于"能操作服务器的 ChatGPT，但跑在 Coze 云端"。

访问方式：通过 Coze 平台使用。

### 前提准备（只需要这两样）

| 需要什么 | 在哪拿到 |
|----------|----------|
| VPS IP | Vultr 服务器详情页 |
| root 密码 | Vultr 服务器详情页，点眼睛图标 |

就这两样，没了。域名、UUID、证书、Caddy……全都不用你管。

### 🤖 无脑版 Prompt

> 📋 把下面这段话里的 IP 和密码换掉，复制发给 Coze的云端OpenClaw：

```
帮我在一台 Vultr VPS 上搭建翻墙代理，要求：

VPS 信息：
- IP：填你的VPS-IP
- root 密码：填你的密码
- 系统：Ubuntu 24.04

要求：
1. 安装 3X-UI 面板，用 VMess + WebSocket + TLS
2. 用 nip.io 自动生成域名，申请/自签 TLS 证书
3. 面板藏在反向代理后面，外面看不出来
4. 当前这台服务器（国内）装上 xray 客户端，连上 VPS
5. 配置好 proxy-on / proxy-off 便捷开关
6. 所有密码、端口、UUID 你自己生成，最后汇总给我
7. 全部自动完成，中间不用问我
```

### Coze的云端OpenClaw 会自动做什么？

| 步骤 | 它自动干什么 | 对应手册章节 |
|------|-------------|-------------|
| 1 | SSH 登录 VPS，更新系统 | 第二步、第三步 |
| 2 | 安装 3X-UI 面板，随机生成端口和密码 | 第四步 |
| 3 | 用 openssl 生成自签 TLS 证书（基于 nip.io） | 第五步方案 B |
| 4 | 在面板里创建 VMess+WS+TLS 入站规则，自动生成 UUID | 第六步 |
| 5 | 当前服务器装 xray 客户端，写配置文件 | 第七步 |
| 6 | 配 proxy-on/proxy-off 别名 | 第七步 7.5 |
| 7 | （可选）装 Caddy 把面板藏起来 | 第八步 |
| 8 | 测试 Google 能通，汇总所有信息给你 | — |

### 📬 你会收到什么？

跑完后 Coze的云端OpenClaw 给你一份汇总：

```
✅ 翻墙代理搭建完成！

VPS 面板：https://1.2.3.4.nip.io/abc123/
面板账号：admin / XXXX

客户端节点 UUID：xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
客户端配置文件：/usr/local/etc/xray-client.json
客户端状态：运行中 ✅

使用方法：
- proxy-on   → 开启代理
- proxy-off  → 关闭代理
- 测试命令  → curl --socks5-hostname 127.0.0.1:10808 https://www.google.com
```

### 如果你没用 Coze的云端OpenClaw

那就翻回上面，从第一步开始手动操作。每一步都写了为什么、在哪查，照着走也能搞定。

### 🔄 进阶：让 Coze的云端OpenClaw 定时检查

搭好之后，可以让 Coze的云端OpenClaw 定期检查代理是否正常：

```
帮我设一个定时任务，每 30 分钟检查一次 xray 客户端是否在运行，
如果挂了自动重启，然后通知我。
```

Coze的云端OpenClaw 会自动创建 cron job，帮你盯着。
