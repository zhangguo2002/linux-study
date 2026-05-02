在 Ubuntu 上通过阿里云镜像安装 Docker，其实就是把 Docker 官方源换成阿里云加速源，这样下载会更快、更稳定。


一、卸载旧版本（如果有）
sudo apt-get remove docker docker-engine docker.io containerd runc
二、安装依赖
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
三、添加阿里云 Docker 源
1. 导入 GPG key
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
2. 添加仓库（阿里云镜像）
sudo mkdir -p /etc/apt/sources.list.d

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
四、安装 Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
五、启动 Docker
sudo systemctl start docker
sudo systemctl enable docker
六、验证是否安装成功
docker --version

测试运行：

sudo docker run hello-world
七（推荐）：配置阿里云镜像加速器（拉镜像更快）

编辑配置文件：

sudo mkdir -p /etc/docker
sudo nano /etc/docker/daemon.json

写入（需要你在阿里云获取专属加速地址）：


{
     "registry-mirrors": [
        "https://docker.1ms.run",
        "https://docker.xuanyuan.me"
    ]
}


获取地址：登录阿里云 → 容器镜像服务 → 镜像加速器

然后重启：

sudo systemctl daemon-reexec
sudo systemctl restart docker