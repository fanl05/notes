# Docker Compose
[Docker Compose](https://www.runoob.com/docker/docker-compose.html)
## 简介
Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。
## 使用步骤
1. 使用 Dockerfile 定义应用程序的环境
2. 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行
3. 执行 docker-compose up 命令来启动并运行整个应用程序
## 安装
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/${version}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
添加可执行权限
```bash
sudo chmod +x /usr/local/bin/docker-compose
```
添加软链接
```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
测试是否安装成功
```bash
docker-compose --version
```