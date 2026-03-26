+++
date = '2026-02-09T15:00:00+08:00'
draft = false
title = 'PairDrop 部署指南'
+++

# PairDrop 自托管部署指南

本指南详细记录了 PairDrop 自托管服务的完整部署流程，涵盖从基础环境搭建到生产环境配置的各个步骤。通过遵循本指南，您可以在 VPS 服务器上成功部署 PairDrop，实现局域网外的文件传输功能。

## 目录

- [环境准备与前置条件](#环境准备与前置条件)
- [Docker 环境安装](#docker-环境安装)
- [PairDrop 容器部署](#pairdrop-容器部署)
- [Caddy 反向代理配置](#caddy-反向代理配置)
- [WebSocket 配置详解](#websocket-配置详解)
- [SSL 证书配置](#ssl-证书配置)
- [Docker 网络配置](#docker-网络配置)
- [防火墙与端口设置](#防火墙与端口设置)
- [环境变量优化](#环境变量优化)
- [调试与日志分析](#调试与日志分析)
- [常见问题与解决方案](#常见问题与解决方案)

---

## 环境准备与前置条件

### 服务器要求

部署 PairDrop 建议使用具有公网 IP 的 VPS 服务器，最低配置要求如下：CPU 至少 1 核心，内存不少于 1GB，硬盘空间 5GB 以上即可。操作系统推荐使用 Ubuntu 20.04/22.04 或 Debian 11/12，这些系统对 Docker 的支持最为完善，社区资料也最为丰富。

### 域名准备

PairDrop 的 WebSocket 功能需要通过 HTTPS 访问，因此您需要准备一个已备案的域名（针对中国大陆服务器）。域名需要在 DNS 服务商处添加 A 记录，指向您的服务器公网 IP。如果您使用 Cloudflare 等 CDN 服务，需要将 DNS 记录设置为 DNS Only 模式（灰色云朵）或 Proxied 模式（橙色云朵），两种模式均可正常工作，但需要注意 SSL 证书的配置方式有所不同。

### 端口要求

服务器防火墙需要开放以下端口：80 端口用于 Caddy 自动申请 Let's Encrypt 证书（HTTP 挑战），443 端口用于 HTTPS 访问和 WebSocket 连接。其他端口如需远程管理，可以根据实际需求开放 SSH（默认 22）、Caddy 管理界面（2019）等。

---

## Docker 环境安装

### 安装 Docker

首先更新系统软件包索引，执行以下命令安装基础依赖：

```bash
sudo apt update && sudo apt upgrade -y
```

安装 Docker 的推荐方式是使用官方安装脚本，该脚本会自动检测系统环境并完成安装：

```bash
curl -fsSL https://get.docker.com | sh -s -- --mirror Aliyun
```

安装完成后，将当前用户添加到 docker 用户组，这样无需 sudo 即可执行 Docker 命令：

```bash
sudo usermod -aG docker $USER
```

执行完上述命令后，建议重新登录 shell 会话以使组成员资格生效。验证 Docker 是否正确安装，可以运行：

```bash
docker --version
docker compose version
```

### 配置 Docker 镜像加速（可选）

中国大陆用户可以使用国内镜像加速器来提高 Docker 镜像的下载速度。编辑 Docker 配置文件：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com"
  ]
}
EOF
```

重启 Docker 服务使配置生效：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

---

## PairDrop 容器部署

### 创建项目目录

为了便于管理，建议在服务器上创建一个专门用于 PairDrop 的目录结构：

```bash
sudo mkdir -p /opt/pairdrop
cd /opt/pairdrop
```

### 编写 Docker Compose 配置文件

在项目目录下创建 docker-compose.yml 文件，这是部署 PairDrop 容器的核心配置：

```bash
sudo nano /opt/pairdrop/docker-compose.yml
```

文件内容如下：

```yaml
version: "3.8"

services:
  pairdrop:
    image: ghcr.io/schlagmichdoch/pairdrop:latest
    container_name: pairdrop
    restart: unless-stopped
    environment:
      - WS_FALLBACK=false
      - DEBUG_MODE=false
    networks:
      - pairdrop-network

networks:
  pairdrop-network:
    name: pairdrop-network
    driver: bridge
```

上述配置说明如下：image 指定使用 GitHub Container Registry 上的最新 PairDrop 镜像；restart 策略设为 unless-stopped，确保容器在服务器重启后自动启动；WS_FALLBACK 设置为 false，用于在排查问题时暴露 WebSocket 相关错误，生产环境可根据需要调整；DEBUG_MODE 设为 false 以减少日志输出，调试时可改为 true。

### 启动 PairDrop 容器

执行以下命令拉取镜像并启动容器：

```bash
cd /opt/pairdrop
sudo docker compose up -d
```

验证容器是否正常运行：

```bash
sudo docker ps | grep pairdrop
```

如果容器状态显示为 Up 且没有 Restarting 字样，说明 PairDrop 基础服务已成功启动。此时可以通过服务器内网 IP 的默认端口（需要查看镜像文档）访问服务，验证其基本功能是否正常。

---

## Caddy 反向代理配置

PairDrop 需要通过 WebSocket 进行实时通信，而 WebSocket 依赖 HTTP 协议升级机制。使用 Caddy 作为反向代理是推荐方案，因为它能自动处理 SSL 证书且配置简洁。

### 安装 Caddy

Caddy 的安装同样推荐使用官方安装脚本：

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

安装完成后，Caddy 服务会自动启动并监听 80 和 443 端口。

### 创建 Caddy 配置目录

```bash
sudo mkdir -p /opt/pairdrop/caddy
cd /opt/pairdrop/caddy
```

### 编写 Caddyfile 配置

创建 Caddyfile，这是 Caddy 的核心配置文件：

```bash
sudo nano /opt/pairdrop/caddy/Caddyfile
```

对于使用 Cloudflare 的用户，推荐配置如下（请将 your-domain.com 替换为您的实际域名）：

```
your-domain.com {
    reverse_proxy localhost:3000

    @websockets {
        header Connection *Upgrade*
        header Upgrade websocket
        path /server
    }

    reverse_proxy @websockets localhost:3000
}
```

上述配置的含义如下：reverse_proxy localhost:3000 将 HTTPS 请求转发到本地的 PairDrop 服务；@websockets 定义了一个命名匹配器，用于识别 WebSocket 连接请求；header 指令设置了 WebSocket 协议升级所需的头部信息；path /server 指定只对 /server 路径的请求应用 WebSocket 转发规则。

如果您不使用 Cloudflare 或希望 Caddy 自动申请 Let's Encrypt 证书，可以使用以下简化配置：

```
your-domain.com {
    reverse_proxy localhost:3000

    @websockets {
        header Connection *Upgrade*
        header Upgrade websocket
        path /server
    }

    reverse_proxy @websockets localhost:3000
}
```

### Caddy 与 Docker 容器集成

由于 PairDrop 运行在 Docker 容器中，而 Caddy 可能运行在宿主机上或另一个容器中，需要确保两者之间能够正常通信。如果 Caddy 也使用 Docker 部署，需要创建共享网络并正确配置容器间的通信。

---

## WebSocket 配置详解

WebSocket 是 PairDrop 实现设备发现和文件传输的核心技术，正确的 WebSocket 配置是部署成功的关键。

### WebSocket 协议升级原理

WebSocket 是一种在单个 TCP 连接上提供全双工通信的协议。与传统 HTTP 请求-响应模式不同，WebSocket 允许服务器主动向客户端推送数据。协议升级过程如下：客户端发送带有 Upgrade 头部的 HTTP 请求；服务器响应 101 Switching Protocols 状态码；连接从 HTTP 切换为 WebSocket，之后的所有通信都使用 WebSocket 帧格式。

### 反向代理 WebSocket 配置要点

在 Nginx 中的关键配置如下：

```nginx
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

在 Caddy 中的等效配置如下：

```caddy
@websockets {
    header Connection *Upgrade*
    header Upgrade websocket
    path /server
}

reverse_proxy @websockets localhost:3000
```

两种配置的核心要点一致：设置 Upgrade 头部为 websocket，设置 Connection 头部为 upgrade，添加 X-Forwarded-For 和 X-Forwarded-Proto 头部以传递客户端真实信息。

### 常见 WebSocket 连接问题

WebSocket 连接失败通常表现为浏览器控制台报错 "WebSocket connection failed" 或 "无法建立到 wss://xxx/server 的连接"。排查方向包括：检查反向代理是否正确添加了 Upgrade 和 Connection 头部；验证后端服务是否在监听正确端口；确认防火墙允许 WebSocket 握手请求通过；检查 SSL 证书是否有效，自签名证书可能导致浏览器阻止连接。

---

## SSL 证书配置

HTTPS 是现代 Web 应用的标配，PairDrop 的 WebSocket 功能要求使用 WSS（WebSocket Secure）协议，这依赖有效的 SSL 证书。

### Let's Encrypt 自动证书

Caddy 默认会自动从 Let's Encrypt 申请和管理 SSL 证书，这是最简单的方式。前提条件是域名 DNS 已正确解析到服务器 IP，且 80 端口未被占用。首次访问域名时，Caddy 会自动完成证书申请过程。

### Cloudflare 源证书

如果您的域名使用 Cloudflare CDN，可以选择使用 Cloudflare 提供的源证书。源证书由 Cloudflare 签发，只能用于 Cloudflare 与源服务器之间的加密通信。在 Cloudflare 控制面板的 SSL/TLS -> 源服务器证书中生成源证书，将证书和私钥保存到服务器，然后在 Caddyfile 中指定证书路径。

### 自签名证书（仅用于测试）

自签名证书不推荐用于生产环境，但如果仅用于内网测试，可以通过以下方式生成：

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /opt/pairdrop/caddy/selfsigned.key \
    -out /opt/pairdrop/caddy/selfsigned.crt \
    -subj "/C=CN/ST=State/L=City/O=Organization/CN=your-domain.com"
```

在 Caddyfile 中引用自签名证书：

```
your-domain.com {
    tls /etc/caddy/certs/selfsigned.crt /etc/caddy/certs/selfsigned.key
    reverse_proxy localhost:3000
}
```

使用自签名证书时，用户首次访问会看到浏览器的安全警告，需要手动信任证书才能正常使用。

---

## Docker 网络配置

Docker 网络配置是容器间通信的基础，正确的网络设置能避免很多连接问题。

### 创建自定义 Docker 网络

推荐为 PairDrop 创建一个专用的桥接网络，这样便于管理和隔离：

```bash
sudo docker network create pairdrop-network
```

### 修改 docker-compose.yml

更新配置文件以使用指定网络：

```yaml
version: "3.8"

services:
  pairdrop:
    image: ghcr.io/schlagmichdoch/pairdrop:latest
    container_name: pairdrop
    restart: unless-stopped
    environment:
      - WS_FALLBACK=false
      - DEBUG_MODE=false
    networks:
      - pairdrop-network
    ports:
      - "127.0.0.1:3000:3000"

networks:
  pairdrop-network:
    external: true
    name: pairdrop-network
```

关键是 ports 配置中的 127.0.0.1 前缀，这只允许本地访问，防止端口暴露到公网。所有外部访问都通过反向代理进行。

### Caddy Docker 容器配置

如果 Caddy 也运行在 Docker 中，需要确保它与 PairDrop 在同一网络：

```yaml
version: "3.8"

services:
  pairdrop:
    image: ghcr.io/schlagmichdoch/pairdrop:latest
    container_name: pairdrop
    restart: unless-stopped
    environment:
      - WS_FALLBACK=false
    networks:
      - pairdrop-network
    expose:
      - "3000"

  caddy:
    image: caddy:2-alpine
    container_name: pairdrop-caddy
    restart: unless-stopped
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./data:/data
      - ./config:/config
    networks:
      - pairdrop-network
    ports:
      - "80:80"
      - "443:443"
      - "2019:2019"

networks:
  pairdrop-network:
    external: true
```

---

## 防火墙与端口设置

### UFW 防火墙配置

如果服务器使用 UFW 防火墙，需要开放 HTTP 和 HTTPS 端口：

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 如果需要 SSH 管理
sudo ufw allow 22/tcp

# 如果需要 Caddy 管理界面
sudo ufw allow 2019/tcp

sudo ufw enable
sudo ufw status verbose
```

### iptables 防火墙配置

对于使用 iptables 的服务器：

```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -L -n
```

### 云服务器安全组设置

如果使用阿里云、腾讯云、AWS 等云服务器，还需要配置云平台的安全组规则，放行 80 和 443 端口的入站流量。

---

## 环境变量优化

PairDrop 支持多个环境变量来控制其行为，合理的配置能提高服务的稳定性和可调试性。

### 常用环境变量说明

WS_FALLBACK 用于控制 WebSocket 降级模式。当设置为 false 时，如果 WebSocket 连接失败，用户会看到明确的错误提示，这有助于排查问题。设置为 true 时，系统会自动降级到轮询等替代方案，但这会牺牲实时性和性能，生产环境建议保持 false。

DEBUG_MODE 用于启用调试模式。当设置为 true 时，PairDrop 会输出详细的日志信息，包括 WebSocket 连接详情、客户端信息、错误堆栈等。这些信息对于排查问题非常有价值，但会增加日志量，生产环境建议保持 false。

### 生产环境推荐配置

```yaml
environment:
  - WS_FALLBACK=false
  - DEBUG_MODE=false
```

### 调试阶段配置

排查问题时可以临时启用调试模式：

```yaml
environment:
  - WS_FALLBACK=false
  - DEBUG_MODE=true
```

查看容器日志：

```bash
sudo docker logs -f pairdrop
```

---

## 调试与日志分析

### 查看容器日志

查看 PairDrop 容器日志的基本命令：

```bash
sudo docker logs pairdrop
sudo docker logs --tail 100 pairdrop
sudo docker logs -f pairdrop
```

-f 参数会持续跟踪日志输出，类似于 tail -f 的效果。Ctrl+C 可以退出跟踪模式。

### 查看 Caddy 日志

如果 Caddy 作为 Docker 容器运行：

```bash
sudo docker logs pairdrop-caddy
sudo docker logs --tail 100 pairdrop-caddy
```

查看 Caddy 日志文件（如果配置了文件日志）：

```bash
sudo tail -f /var/log/caddy/access.log
sudo tail -f /var/log/caddy/error.log
```

### 浏览器开发者工具

浏览器开发者工具是排查 WebSocket 问题的利器。在 Firefox 或 Chrome 中按 F12 打开开发者工具，切换到「网络」标签，勾选「过滤 WebSocket」或输入 ws:// 或 wss:// 筛选，然后刷新页面观察 WebSocket 连接请求。

「控制台」标签会显示 WebSocket 连接的错误信息，如 "Firefox 无法建立到 wss://xxx/server 的连接"，这些信息是定位问题的关键线索。

### 网络抓包分析

对于复杂问题，可以使用 tcpdump 或 Wireshark 进行网络抓包：

```bash
sudo tcpdump -i any -n port 443 -w /tmp/https.pcap
```

然后使用 Wireshark 分析抓包文件，重点关注 TLS 握手和 WebSocket 升级过程。

---

## 常见问题与解决方案

### 问题一：WebSocket 连接失败

现象：浏览器控制台报错 "WebSocket connection failed"，页面显示"连接中"。

原因：反向代理未正确配置 WebSocket 协议升级。

解决：确保反向代理配置包含以下头部：

- Upgrade: websocket
- Connection: upgrade

参考本文档「WebSocket 配置详解」章节的配置示例。

### 问题二：Caddy 容器持续重启

现象：docker ps 显示容器状态为 Restarting，docker logs 显示错误信息。

原因：常见原因包括 Caddyfile 语法错误、证书文件路径错误、Docker 网络不存在等。

解决：按以下步骤排查：

1. 检查 Caddyfile 语法是否正确，文件路径是否使用了正确的绝对路径。
2. 如果使用自定义证书，确认证书文件存在且权限正确。
3. 确认 Docker 网络已创建，使用 docker network ls 检查。
4. 查看完整错误日志，docker logs --tail 200 pairdrop-caddy。

### 问题三：Let's Encrypt 证书申请失败

现象：Caddy 日志显示 "certificate provisioning for domain failed"。

原因：域名 DNS 未正确解析到服务器 IP、80 端口被其他服务占用、Let's Encrypt 频率限制等。

解决：

1. 确认域名 DNS 已生效，使用 ping 或 nslookup 验证。
2. 检查 80 端口是否有其他服务占用，sudo lsof -i :80。
3. 等待一段时间后重试，Let's Encrypt 有频率限制。
4. 查看 Caddy 日志获取详细的错误原因。

### 问题四：Cloudflare 521 错误

现象：通过 Cloudflare 访问时返回 521 错误码。

原因：Cloudflare 无法连接到源服务器，可能是因为源服务器防火墙阻止了 Cloudflare IP 段。

解决：

1. 确认服务器防火墙允许所有来源的 80/443 入站流量。
2. 检查 Cloudflare 源 IP 范围是否被特别限制。
3. 临时将 Cloudflare DNS 设置为 DNS Only 测试。
4. 确认服务器 Web 服务正常运行。

### 问题五：证书 525 SSL 握手失败

现象：Cloudflare SSL 握手失败，错误码 525。

原因：服务器 SSL 配置与 Cloudflare 不兼容，常见于使用 Cloudflare 源证书但配置错误的情况。

解决：

1. 如果使用 Cloudflare 源证书，确认证书链完整。
2. 确认服务器支持 Cloudflare 要求的 TLS 版本（通常需要 TLS 1.2+）。
3. 检查 Caddyfile 中的 tls 配置是否正确。
4. 尝试使用 Caddy 自动申请的 Let's Encrypt 证书。

### 问题六：自签名证书浏览器不信任

现象：页面能打开但 WebSocket 连接失败，浏览器控制台显示证书错误。

原因：浏览器不信任自签名证书。

解决：

1. 导出证书并手动导入到系统受信任根证书颁发机构。
2. 建议改用 Let's Encrypt 免费证书，这是最简单的解决方案。
3. 如果必须使用自签名，确保证书包含正确的域名 CN。

---

## 完整部署检查清单

部署完成后，使用以下清单进行最终验证：

### 基础连通性检查

- [ ] 服务器公网 IP 可以 ping 通
- [ ] 域名 DNS 正确解析到服务器 IP
- [ ] 80 和 443 端口防火墙已开放

### Docker 服务检查

- [ ] Docker 和 Docker Compose 已正确安装
- [ ] pairdrop 容器状态为 Up
- [ ] pairdrop-caddy 容器状态为 Up（如果使用 Docker 部署 Caddy）
- [ ] 容器未出现 Restarting 状态

### 功能验证检查

- [ ] 通过 HTTPS 访问域名可以打开 PairDrop 页面
- [ ] 页面显示"已连接"而非"连接中"
- [ ] 浏览器控制台无 WebSocket 错误
- [ ] 网络标签页可见成功的 WebSocket 连接（状态码 101）

### 安全配置检查

- [ ] 使用有效的 SSL 证书（非自签名用于生产）
- [ ] HTTP 自动重定向到 HTTPS
- [ ] Caddy 管理界面已限制访问或使用强密码

---

## 维护与更新

### 更新 PairDrop

PairDrop 镜像会定期更新，添加新功能和修复问题。更新步骤如下：

```bash
cd /opt/pairdrop
sudo docker compose pull
sudo docker compose up -d
sudo docker image prune -f
```

### 更新 Caddy

```bash
sudo apt update && sudo apt upgrade caddy
sudo systemctl restart caddy
```

### 备份配置

定期备份重要配置文件：

```bash
sudo tar -czvf pairdrop-backup-$(date +%Y%m%d).tar.gz \
    /opt/pairdrop/docker-compose.yml \
    /opt/pairdrop/caddy/Caddyfile \
    /opt/pairdrop/caddy/data \
    /opt/pairdrop/caddy/config
```

---

## 总结

PairDrop 自托管部署的核心要点包括：使用 Docker 容器化部署便于管理；Caddy 反向代理提供简洁的 WebSocket 配置和自动 SSL 证书管理；正确的网络配置确保容器间通信畅通；完善的安全设置保护服务安全。遵循本指南的步骤，您应该能够成功部署一个稳定运行的 PairDrop 服务。

如果在部署过程中遇到本指南未涵盖的问题，欢迎提供详细的错误信息，以便进一步排查和补充文档。
