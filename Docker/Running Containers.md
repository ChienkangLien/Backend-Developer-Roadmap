# Running Containers
* 啟動一個新容器

使用 docker run 命令後接著是image 名稱。基本語法如下：`docker run [options] IMAGE [COMMAND] [ARG...]`

例如，運行官方的 Nginx image：`docker run -d -p 8080:80 nginx`
這會啟動一個新容器並將主機的端口8080映射到容器的端口80。
* 列出容器

使用`docker ps`命令。要查看所有容器（包括已停止的），使用 -a 標誌：`docker container ls -a`
* 訪問容器

要訪問運行中容器的 shell，使用 docker exec 命令：`docker exec -it CONTAINER_ID bash`
將CONTAINER_ID 替換為您想要訪問的容器的ID 或名稱。您可以在`docker ps`的輸出中找到它。

* 停止容器

使用`docker stop`命令後接著是容器的 ID 或名稱：
`docker container stop CONTAINER_ID`
* 移除容器

一旦容器停止，我們可以使用`docker rm`命令後接著是容器的 ID 或名稱來移除它：
`docker container rm CONTAINER_ID`
要在容器退出時自動移除容器，運行容器時加入 --rm 標誌：`docker run --rm IMAGE`

## Runtime Configuration Options
運行時配置選項允許您在運行Docker 容器時自定義其行為和資源。以下是一些常用的配置選項：
* 資源管理

CPU：您可以使用 --cpus 限制容器可使用的CPU核心數量，而 --cpu-shares 分配容器的CPU時間相對份額。
`docker run --cpus=2 --cpu-shares=512 your-image`
內存：您可以使用 --memory 和 --memory-reservation 限制和保留容器的內存。這有助於防止容器佔用過多系統資源。
`docker run --memory=1G --memory-reservation=500M your-image`
* 安全性

用戶：默認情況下，容器以root 用戶運行。為了增強安全性，可以使用 --user 選項以其他用戶或UID運行容器。
`docker run --user 1000 your-image`
唯讀root 檔案系統：為防止對容器檔案系統的不必要更改，可以使用 --read-only 選項將根檔案系統掛載為為讀。
`docker run --read-only your-image`
* 網絡

發布端口：您可以使用 --publish（或 -p）選項將容器的端口發布到主機系統，這允許外部系統訪問容器化的服務。
`docker run -p 80:80 your-image`
主機名和DNS：您可以使用 --hostname 和 --dns 選項自定義容器的主機名和DNS設置。
`docker run --hostname=my-container --dns=8.8.8.8 your-image`

更多有關可用的運行時配置選項的完整列表 https://docs.docker.com/engine/reference/run/。

## Docker Compose
一款定義和運行多容器 Docker 應用程序的工具。使用一個名為 docker-compose.yml 的 YAML 文件來創建、管理和運行應用程序。這個文件描述了應用程序的服務、網絡和數據卷，僅使用一個命令運行和管理容器。
* 簡化容器管理：一個地方定義和配置所有服務、網絡和卷。
* 可重現構建：將您的 docker-compose.yml 文件與其他人分享，確保他們運行與您相同的環境和服務。
* 版本控制：Docker Compose 文件可以進行版本化控制。

### 創建 Docker Compose 文件
首先指定您想要使用的 Docker Compose 版本，然後是您想要定義的服務。
```
version: "3.9"
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
  db:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: mysecretpassword
```
在這個示例中，我們指定了兩個服務：一個名為 web 的 Web 服務器，運行最新版本的 nginx image，以及一個名為 db 的數據庫服務器，運行 MySQL。Web 服務器將其端口 80 映射到主機機器，數據庫服務器設置了一個用於 root 密碼的環境變量。

### 運行 Docker Compose
導航到包含您的 docker-compose.yml 文件的目錄，然後運行以下命令：
`docker-compose up`
Docker Compose 將讀取文件並按照指定的順序啟動定義的服務。

### 其他命令：
* `docker-compose down`：停止並刪除在 docker-compose.yml 文件中定義的所有運行中的容器、網絡和卷。
* `docker-compose ps`：列出在 docker-compose.yml 文件中定義的所有容器的狀態。
* `docker-compose logs`：顯示在 docker-compose.yml 文件中定義的所有容器的日誌。
* `docker-compose build`：構建在 docker-compose.yml 文件中定義的所有image。


更多信息 https://docs.docker.com/compose/ 。

## 使用 docker run 運行容器
docker run 命令的基本語法如下：
`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`
* OPTIONS：這些是命令行標誌，用於調整容器的設置，如內存限制、端口、環境變量等。
* IMAGE：容器將運行的 Docker image。這可以是來自 Docker Hub 或本地存儲的自有image。
* COMMAND：這是容器啟動時將在其中執行的命令。如果未指定，將使用image 的默認入口點（entrypoint）。
* ARG...：這些是可選參數，可以傳遞給要執行的命令。

### 常用選項
* `--name`：為容器分配一個名稱。
* `-p, --publish`：將容器的端口發布到主機。
* `-e, --env`：在容器內設置環境變量。
* `-d, --detach`：以分離模式運行容器，在後台運行容器並且不在控制台顯示日誌。
* `-v, --volume`：將主機上的卷進行綁定掛載到容器。這有助於持久保存容器生成的數據，或在主機和容器之間共享文件。

運行 Ubuntu 容器的交互式會話：
`docker run -it --name=my-ubuntu ubuntu`

運行 Nginx Web 服務器並在主機上發布端口 80：
`docker run -d --name=my-nginx -p 80:80 nginx`

運行 MySQL 容器並使用自定義環境變量配置數據庫：
`docker run -d --name=my-mysql -e MYSQL_ROOT_PASSWORD=secret -e MYSQL_DATABASE=mydb -p 3306:3306 mysql`

使用綁定掛載卷運行容器：
`docker run -d --name=my-data -v /path/on/host:/path/in/container some-image`

## Docker Networks
Docker Networks 提供了管理容器通信的重要方式。它允許容器使用各種網路驅動程序相互通信，並與主機機器進行通信。

### 網路驅動程序
* bridge：容器的默認網路驅動程序。它創建了一個私有網路，容器可以在其中相互通信，也可以通過主機的網路訪問外部資源。
* host：該驅動程序取消了網路隔離，允許容器共享主機的網路；可以減少容器網路的開銷。
* none：該網路驅動程序禁用容器的網路。使用此驅動程序運行的容器在沒有任何網路訪問的隔離環境中運行。
* overlay：該網路驅動程序使部署在不同主機上的容器可以彼此通信。它設計用於與 Docker Swarm 一起使用，非常適合多主機或基於集群的容器部署。

### 管理 Docker 網路
列出所有網路：`docker network ls`
檢視網路：`docker network inspect <network_name>`
創建新網路：`docker network create --driver <driver_type> <network_name>`
將容器連接到網路：`docker network connect <network_name> <container_name>`
從網路斷開容器：`docker network disconnect <network_name> <container_name>`
刪除網路：`docker network rm <network_name>`