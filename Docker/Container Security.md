# Container Security
容器隔離
容器應該相互隔離，並與主機系統隔離，以防止未授權訪問，並減輕攻擊者在成功破壞一個容器時的潛在損害。
* Namespaces：Docker 使用Namespaces 技術為運行容器提供隔離環境。Namespaces 限制容器在系統中所能看到和訪問的範圍，包括進程和網絡資源。
* Cgroups：用於限制容器消耗的資源，如 CPU、內存和 I/O。正確使用Cgroups 有助於防止 DoS 攻擊和資源耗盡情況。

安全模式和實踐
* 最小權限原則：應以最低可能的權限運行容器，僅授予應用所需的最低權限。
* 不可變的基礎設施：應將容器視為不可變的單元，一旦建立，就不應更改。任何更改都應通過使用更新映像部署新容器。
* 版本控制：image 應進行版本控制並存儲在安全的容器註冊庫中。

安全訪問控制
* 容器管理：使用基於角色的訪問控制（RBAC）來限制對容器管理平台（例如 Kubernetes）的訪問，確保用戶僅具有必要的最低權限。
* 容器數據：對靜態和傳輸中的數據進行加密，特別是當處理敏感信息時。

容器漏洞管理
* image 掃描：使用自動掃描工具識別容器和image 中的漏洞，並應集成到開發管道中。
* 安全基礎image：使用最小和安全的基礎image 進行容器創建，減少攻擊面和潛在漏洞。
* 定期更新基礎image 和容器。

## Runtime Security
唯讀檔案系統
將容器的檔案系統設置為唯讀，可以防止攻擊者修改關鍵文件或在容器內植入惡意軟件。
* 在啟動容器時使用 `--read-only` 標誌，使其檔案系統為唯讀。
* 對需要寫入訪問權限的位置實施卷掛載或 `tmpfs` 掛載。

安全掃描與監控
* 使用容器掃描工具來檢測並修補image 中的漏洞。
* 實施運行時監控以檢測和響應安全事件，如未授權訪問嘗試或意外進程啟動。

資源隔離
* 使用 Docker 內建的資源限制來限制容器可以消耗的資源。
* 使用網絡分割和防火墻來隔離容器並限制其通信。

審計日誌
* 使用 Docker 的日誌功能來捕獲容器日誌，將其輸出到集中日誌解決方案。
* 實施日誌分析工具以監控可疑活動，並在檢測到潛在事件時自動發出警報。

## Image Security
使用可信賴的來源並檢查 Dockerfile 和其他提供的文件，以確保它們遵循最佳實踐並且不引入漏洞。

官方image：https://hub.docker.com/explore/

1. 保持image 最新

使用以下工具掃描和檢查image 是否有更新：
Docker Hub：https://hub.docker.com/
Anchore：https://anchore.com/
Clair：https://github.com/quay/clair

2. 使用最小基礎image

僅包含運行容器化應用程序所需的基本內容。組件越少，潛在漏洞的攻擊面就越小。

最小基礎image 的一個示例是 Alpine Linux 發行版，由於其小的佔用空間和安全功能，它在 Docker image 中常被使用。
Alpine Linux：https://alpinelinux.org/

3. 掃描圖像中的漏洞

使用 Clair 或 Anchore 等工具檢測image 和容器配置中的潛在風險，在將image 推送到註冊庫或在生產環境中部署之前解決問題。

4. 對image 進行簽名和驗證

使用 Docker Content Trust（DCT）對其進行簽名。DCT 使用數字簽名來保證拉取或推送的image 是您預期的image，並且在傳輸過程中未被篡改。

通過設置以下環境變量來為您的 Docker 環境啟用 DCT：
`export DOCKER_CONTENT_TRUST=1`

5. 利用多階段構建

在同一個 Dockerfile 中使用多個 FROM 指令。最小化最終image 的大小和複雜性，從而降低潛在漏洞的風險。
```
# Build stage
FROM node:12-alpine AS build
WORKDIR /app
COPY . .
RUN npm ci --production

# Final stage
FROM node:12-alpine
COPY --from=build /app /app
CMD ["npm", "start"]
```
