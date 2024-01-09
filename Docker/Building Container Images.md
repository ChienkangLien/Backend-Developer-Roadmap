# Building Container Images
容器image 是可執行的套件，其中包括運行應用程序所需的一切：代碼、runtime、系統工具、libraries 以及設置。通過構建自定義image，您可以在任何支持Docker 的平台上無縫部署應用程序及其所有依賴項。

## Dockerfile
Dockerfile 是一個文本文檔，包含了一系列指令，用於構建image。每個指令都會為image 添加一個新的層級。Docker 將根據這些指令構建image ，然後您可以從該image 運行容器。

### Dockerfile 的結構
每個指令都有特定的格式：
`INSTRUCTION arguments` 即是一個指令跟著一個參數

以下是一個簡單的示例：
```
# 使用官方的 OpenJDK 映像作為基礎映像
FROM openjdk:11

# 設置工作目錄為 /app
WORKDIR /app

# 複製打包好的 Java 可執行 JAR 檔案到容器中
COPY target/my-app.jar /app/my-app.jar

# 安裝所需的軟體依賴（這裡示範安裝 curl）
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# 設定環境變數
ENV APP_VERSION=1.0.0

# 暴露應用程序運行的端口（如果需要的話）
EXPOSE 8080

# 定義默認的運行命令
CMD ["java", "-jar", "my-app.jar"]
```
常見的 Dockerfile 指令：

* FROM：設置用於開始的基本image。在 Dockerfile 中，FROM 必須是第一條指令。
* WORKDIR：設置任何 RUN、CMD、ENTRYPOINT、COPY 或 ADD 指令的工作目錄。如果目錄不存在，將自動創建。
* COPY：將主機上的文件或目錄複製到容器的文件系統中。
* ADD：類似於 COPY，但也可以處理遠程 URL，並自動解壓存檔文件。
* RUN：在image 中作為新層執行命令。
* CMD：定義從image 運行容器時的默認命令。
* ENTRYPOINT：與 CMD 類似，但設計為允許容器作為帶有自己參數的可執行文件。
* EXPOSE：通知 Docker 容器在運行時會聽取指定的網絡端口。
* ENV：為容器設置環境變量。


### 構建image
使用 docker build 命令，指定構建上下文（通常為當前目錄），以及image 的可選標籤。

`docker build -t my-app-image:1.0 .` 在當前目錄中尋找 Dockerfile，然後使用它來構建一個新的 image，並且將其命名為 my-app-image，版本號為 1.0。

* -f 選項可以用於指定 Dockerfile 文件的名稱或路徑。
* --tag 或 -t 選項可以用於指定 image 的名稱和標籤。

例如，要使用另一個名稱或路徑的 Dockerfile：
`docker build -f /path/to/Dockerfile -t my-app-image:1.0 .`
或者，要添加額外的標籤：
`docker build -t my-app-image:1.0 -t my-app-image:latest .`
這會創建兩個標籤為 my-app-image:1.0 和 my-app-image:latest 的 image。

## Efficient Layer Caching
在構建容器image 時，Docker 會對新創建的層進行緩存。這些層稍後可以在構建其他image 時使用，從而減少構建時間並最小化頻寬使用。

### 層緩存的運作方式
Docker 為 Dockerfile 中的每個指令（例如 RUN、COPY、ADD 等）創建一個新的層。如果自上次構建以來該指令沒有更改，Docker 將重用現有的層。

例如，考慮以下 Dockerfile：
```
FROM node:14

WORKDIR /app

COPY package.json /app/
RUN npm install

COPY . /app/

CMD ["npm", "start"]
```
第一次構建image 時，Docker 將執行每個指令並為每個指令創建一個新的層。如果對應用程序進行更改並再次構建image，Docker 將檢查更改的指令是否影響任何層。如果沒有層受到更改的影響，Docker 將重用緩存的層。

### 高效使用層緩存的技巧
* 最小化 Dockerfile 中的更改：嘗試減少 Dockerfile 中的更改頻率，並將最常更改的行出現在底部的方式組織指令。
* 構建上下文優化：使用 .dockerignore 文件排除構建上下文中不必要的文件，這些文件可能會導致緩存失效。
* 使用更小的基本image：較小的基本image 減少了拉取基本image 所需的時間，以及需要緩存的層數量。
* 充分利用 Docker 的`--cache-from` 標誌：如果您使用 CI/CD 流水線，可以指定要用作緩存來源的image。
* 結合多個指令：在某些情況下，結合指令（例如`RUN`）有助於減少層數，從而使緩存更有效。

## Image Size and Security
較小的image 導致更快的構建速度，並在下載時減少網絡開銷。安全性很重要，因為容器image 可能包含潛在的漏洞。

### 減小image 大小
* 選擇適當的基本image：選擇一個較小、更輕量的基本image，其中僅包括應用程序所需的必要組件。例如，考慮使用官方的 alpine 變體，因為它通常體積更小。
`FROM node:14-alpine`
* 在單個 RUN 語句中運行多個命令：每個 RUN 語句在image 中創建一個新的層，這將影響大小。使用 && 將多個命令組合成單個 RUN 語句，以最小化層數並減小最終大小。
```
RUN apt-get update && \
    apt-get install -y some-required-package
```
* 在同一層中刪除不必要的文件：在構建過程中安裝包或添加文件時，在同一層中刪除臨時或未使用的文件，以減小最終大小。
```
RUN apt-get update && \
    apt-get install -y some-required-package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```
* 使用多階段構建：多階段構建允許在 Dockerfile 中使用多個 FROM 語句。每個 FROM 語句在構建過程中創建一個新的階段。您可以使用 COPY --from 語句將文件從一個階段複製到另一個階段。
```
FROM node:14-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install  #安裝應用程序的依賴項
COPY . .  #複製應用程序的所有文件到 /app 目錄中
RUN npm run build  #運行構建腳本，例如編譯 TypeScript 或打包 React 等

FROM node:14-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist  #從第一階段（build 階段）中的 /app/dist 複製構建後的產物到此階段的 /app/dist 目錄中
COPY package*.json ./
RUN npm install --production  #只安裝生產環境所需的依賴項
CMD ["npm", "start"]  #設置容器啟動時的默認命令
```
第一個階段（Stage）使用了 node:14-alpine 作為基本image，命名為 build。這個階段用於構建和編譯應用程序。

第二個階段不再使用 AS build，因此沒有命名。它基於相同的 node:14-alpine 基本映像，但是此階段只包含構建後的產物，而不包含開發時的所有依賴項。
* 使用 .dockerignore 文件：從構建上下文中排除可能導致緩存失效並增加最終image 大小的不必要文件。
```
node_modules
npm-debug.log
```
### 增強安全性
* 保持基本image 更新：定期更新 Dockerfile 中使用的基本image，以確保其包含最新的安全補丁。
* 避免以 root 運行容器：始終在運行容器時使用非 root 用戶以減少潛在風險。在運行應用程序之前創建一個用戶並切換到該用戶。
```
RUN addgroup -g 1000 appuser && \
    adduser -u 1000 -G appuser -D appuser
USER appuser
```
* 限制 `COPY` 或 `ADD` 指令的範圍：對要複製到容器image 中的文件或目錄進行明確規定。避免使用 `COPY . .`，因為它可能無意中包含敏感文件。
```
COPY package*.json ./
COPY src/ src/
```
* 掃描image 以查找漏洞：使用 Anchore 或 Clair 等工具掃描image 以查找漏洞並在部署之前進行修復。