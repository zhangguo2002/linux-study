# 方式一（HTTPS）：Caddy + Nginx 自动化 HTTPS 架构部署手册
本手册用于配置 Caddy (前置网关/自动加密) + Nginx (后端业务/静态资源) 的双服务器架构。

## 1. 架构逻辑
流量入口：用户 -> Cloudflare (Full Strict) -> Caddy (端口 80/443)。

证书管理：Caddy 自动申请 SSL 证书并处理 HTTPS。

业务处理：Caddy 将流量通过反向代理转发至本地 Nginx (端口 8080)。

## 2. 安装组件
2.1 安装 Caddy
Bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy -y
2.2 安装 Nginx (如果已安装则跳过)
Bash
sudo apt update
sudo apt install nginx -y
## 3. 配置 Nginx (业务层)
修改 Nginx 配置，使其不再监听公网 80 端口，而是退居后台 8080 端口。

编辑站点配置文件：
sudo nano /etc/nginx/sites-available/yinmatech

写入以下配置：

Nginx
server {
    # 核心：仅监听本地回环地址的 8080 端口
    listen 127.0.0.1:8080;

    server_name yinmatech.online www.yinmatech.online;

    root /var/www/yinmatech;
    index index.html;

    location / {
        # 传递真实 IP 的配置
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        try_files $uri $uri/ =404;
    }
}
移除默认冲突配置：
sudo rm /etc/nginx/sites-enabled/default

测试并启动：

Bash
    sudo nginx -t
    sudo systemctl restart nginx
    sudo systemctl enable nginx
    ```

---

## 4. 配置 Caddy (HTTPS 门卫)

1.  **编辑 Caddyfile**：
    `sudo nano /etc/caddy/Caddyfile`
2.  **清空内容，覆盖为**：
    ```caddyfile
    yinmatech.online, www.yinmatech.online {
        # 将流量反向代理给本地 Nginx
        reverse_proxy 127.0.0.1:8080
    }
    ```
3.  **强力清理端口占用并启动**：
    ```bash
    # 安装清理工具
    sudo apt install psmisc -y
    # 强制杀死占用 80/443 的残余进程
    sudo fuser -k 80/tcp
    sudo fuser -k 443/tcp
    # 启动 Caddy
    sudo systemctl start caddy
    sudo systemctl enable caddy
    ```

---

## 5. 运维常用命令

| 任务 | 命令 |
| :--- | :--- |
| **查看 Caddy 运行状态** | `systemctl status caddy` |
| **查看 Caddy 实时日志** | `journalctl -u caddy -f` |
| **重载 Caddy 配置** | `sudo systemctl reload caddy` |
| **检查 Nginx 配置** | `sudo nginx -t` |
| **检查本地端口占用** | `sudo ss -tulpn | grep -E ':80|:443|:8080'` |

---

## 6. Cloudflare 配置要点

1.  **DNS 设置**：确认 `yinmatech.online` 的 A 记录指向服务器公网 IP。
2.  **SSL/TLS 模式**：
    *   点击 **Configure**。
    *   选择 **Full (strict)** 模式。
3.  **安全组**：确认云服务器后台防火墙已开放 **80** 和 **443** 端口。

---
**配置完成！** 现在你的站点已具备：全自动证书续期、双服务器安全隔离、以及高性能静态资源处理能力。


# 方式二（HTTP）：Nginx 基础 HTTP 部署手册
本手册用于配置一个标准的 Nginx 服务，通过 80 端口 提供普通的 HTTP 访问。

1. 安装 Nginx
在 Ubuntu/Debian 系统上执行：

Bash
sudo apt update
sudo apt install nginx -y
2. 准备网页目录
确保你的静态文件已经放在对应的目录下，并设置好权限。

Bash
# 创建网站根目录
sudo mkdir -p /var/www/yinmatech

# 设置权限（让 nginx 用户可以读取）
sudo chown -R www-data:www-data /var/www/yinmatech
sudo chmod -R 755 /var/www/yinmatech
3. 配置 Nginx 虚拟主机
创建新的配置文件：
sudo nano /etc/nginx/sites-available/yinmatech

粘贴以下标准 HTTP 配置：

Nginx
server {
    # 监听标准 HTTP 80 端口
    listen 80;
    listen [::]:80;

    # 你的域名或服务器 IP
    server_name yinmatech.online www.yinmatech.online;

    # 网页文件存放路径
    root /var/www/yinmatech;
    index index.html;

    location / {
        # 尝试按顺序寻找文件，找不到则返回 404
        try_files $uri $uri/ =404;
    }

    # 开启 Gzip 压缩（可选，优化加载速度）
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
}
4. 启用配置并启动
建立配置文件软链接（启用该站点）：

Bash
sudo ln -s /etc/nginx/sites-available/yinmatech /etc/nginx/sites-enabled/
移除默认的 "Welcome to nginx" 配置（防止冲突）：

Bash
sudo rm /etc/nginx/sites-enabled/default
检查语法并重启：

Bash
sudo nginx -t
sudo systemctl restart nginx
设置开机自启：

Bash
sudo systemctl enable nginx


---

## 5. 运维管理命令

| 任务 | 命令 |
| :--- | :--- |
| **启动 Nginx** | `sudo systemctl start nginx` |
| **停止 Nginx** | `sudo systemctl stop nginx` |
| **检查配置是否正确** | `sudo nginx -t` |
| **修改配置后重载** | `sudo systemctl reload nginx` |
| **查看访问日志** | `tail -f /var/log/nginx/access.log` |
| **查看错误日志** | `tail -f /var/log/nginx/error.log` |

---

## 6. 注意事项

1.  **端口冲突**：如果你的服务器上安装了 Caddy 且正在运行，Nginx 将无法启动。请先执行 `sudo systemctl stop caddy`。
2.  **防火墙**：请确保你的云服务器后台（安全组）已经放行了 **80 端口**。
3.  **Cloudflare 用户**：如果你在 Cloudflare 后台开启了小云朵（Proxy），此时 Cloudflare 的 SSL/TLS 模式必须设置为 **Off** 或 **Flexible**（不推荐，因为这不安全），否则浏览器会因为服务器不支持加密而报错。