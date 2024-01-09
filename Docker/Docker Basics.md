# Docker Basics
* Docker Engine：它是Docker的核心部分，是用於構建和運行容器的後台服務。Docker Engine負責管理容器的生命周期、鏡像的構建、存儲和分發。它包括dockerd服務，是在操作系統上實際運行容器的進程。Docker Engine可以在Linux、Windows和macOS等不同操作系統上運行。
* Docker Desktop：這是Docker Engine的桌面版，旨在讓開發人員在他們的個人計算機上更輕松地構建和測試容器化應用程序。Docker Desktop提供了一個簡化的方式來安裝和管理Docker Engine，還包括了Docker CLI、Compose和其他與容器相關的工具。它為開發人員提供了在本地環境中快速構建、測試和部署容器化應用程序的能力。Docker Desktop支持Windows和macOS操作系統。
## Docker 組件
Docker 生態系統中有三個關鍵組件：

1. Dockerfile：一個文本文件，包含構建 Docker image的指令（命令）。
2. Docker image：從 Dockerfile 創建的容器快照。image 存儲在像 Docker Hub 這樣的注冊表中，可以從注冊表拉取或推送。
3. Docker 容器：一個運行中的 Docker image 實例。

## 安裝Docker Engine
CentOS
* `sudo yum install -y yum-utils` 安裝yum-utils軟體包（提供yum-config-manager 工具）
* `sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo` 設定存儲庫
* `sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin` 安裝最新版本
* `sudo systemctl start docker` 啟動

## Docker 命令
以下是一些常用的 Docker 命令：
* `docker pull <image>`：從注冊表（比如 Docker Hub）下載鏡像。
* `docker build -t <image_name> <path>`：從 Dockerfile 構建一個鏡像，其中 \<path> 是包含 Dockerfile 的目錄。
* `docker image ls`：列出本地計算機上所有可用的鏡像。
* `docker run -d -p <host_port>:<container_port> --name <container_name> <image>`：從image 運行一個容器，將主機端口映射到容器端口。
* `docker container ls`：列出所有正在運行的容器。
* `docker container stop <container>`：停止一個正在運行的容器。
* `docker container rm <container>`：刪除一個已停止的容器。
* `docker image rm <image>`：從本地計算機中刪除一個image。