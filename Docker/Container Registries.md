# Container Registries
用於存儲和分發 Docker 容器image 的集中式系統。
而Docker Hub由Docker Inc提供、是Docker image 的默認使用倉庫。

## Image Tagging Best Practices
對 Docker image 進行正確的標記。

1. 使用語義化版本號
在標記image 時，建議遵循語義化版本號(Semantic Versioning guidelines https://semver.org/)的指南。標準的版號必須（MUST）採用 X.Y.Z 的格式，其中 X、Y 和 Z 為非負的整數，且禁止（MUST NOT）在數字前方補零。X 是主版號、Y 是次版號、而 Z 為修訂號。每個元素必須（MUST）以數值來遞增。例如：1.9.1 -> 1.10.0 -> 1.11.0。
2. 標記最新版本
Docker 允許您除了版本號外，將image 標記為 'latest'。常見做法是將最新的穩定版本image 標記為 'latest'。
`docker build -t your-username/app-name:latest .`
3. 說明性和一致性
選擇清晰且具有描述性的標籤名稱，以傳達image 的用途或與先前版本相比的變更，並與存儲庫中保持一致。
4. 包含構建和 Git 資訊（可選）
在某些情況下，將構建和 Git 提交的信息包含在標籤中可能會有所幫助。幫助識別用於構建image 的源代碼和環境。例如：app-name-1.2.3-b567-d1234efg。
5. 使用環境和特定架構的標籤
如果應用程序在不同環境（生產、預發、開發）部署，或具有多個架構（amd64、arm64），可以使用指定這些變化的標籤。例如：your-username/app-name:1.2.3-production-amd64。
6. 使用自動化構建和標記工具
考慮使用 CI/CD 工具（Jenkins、GitLab CI、Travis-CI）根據提交、分支或其他規則自動進行image 構建和標記。

## DockerHub
* 公共和私有存儲庫：存儲在公共存儲庫中，可供所有人訪問，或者選擇私有存儲庫，僅限您的團隊或組織訪問。
* 自動構建：DockerHub 與流行的代碼存儲庫（如GitHub和Bitbucket）集成，允許您為Docker image 設置自動構建。每當您推送代碼到存儲庫時，DockerHub 將自動創建帶有最新更改的image。
* Webhooks：DockerHub 允許您配置Webhooks，在image 建立或更新時通知其他應用程序或服務。
* 組織和團隊：通過創建組織和團隊來輕鬆進行協作，以管理對您的image 和存儲庫的訪問權限。
* 官方image：DockerHub提供了一組經過策劃的官方image，用於流行軟件，如MongoDB、Node.js、Redis等。

當您準備分享自己的image 時，使用docker命令行工具將本地image 推送到DockerHub：
```
docker login
docker tag your-image your-username/your-repository:your-tag
docker push your-username/your-repository:your-tag
```
要從DockerHub 拉取image，使用docker pull命令：
`docker pull your-username/your-repository:your-tag`


## DockerHub Alternatives
* Quay.io
是 Red Hat 推出的，提供免費和付費計劃。它提供一個先進的安全功能稱為「容器安全掃描」，可檢查存儲在您存儲庫中的image 的漏洞。還提供自動構建、細粒度的用戶訪問控制和 Git 存儲庫集成等功能。
* Google Container Registry (GCR)
是 Google Cloud Platform 提供的，GCR 提供與其他 Google Cloud 服務的集成，例如 Cloud Build 用於自動構建、Container Registry 漏洞掃描以及 IAM 角色用於用戶訪問控制。
* Amazon Elastic Container Registry (ECR)
由 Amazon Web Services（AWS）提供，可以使用 AWS 身份和訪問管理（IAM）策略控制對image 的訪問。ECR 還與其他 AWS 服務集成，例如 Lambda、Amazon ECS 和 ECR image 掃描。
* Azure Container Registry (ACR)
由 Microsoft Azure 提供，提供多種功能，包括用於高可用性的地理複寫、ACR 任務用於自動image 構建、漏洞掃描以及與 Azure Pipelines 用於 CI/CD 的集成。ACR 還提供使用虛擬網路和防火牆進行私有網路訪問。
* GitHub Container Registry (GHCR)
由GitHub 提供，它提供更簡化的體驗來增強 GitHub Packages 中 Docker 的支持，用於管理和部署 Docker image。GHCR 提供細粒度的訪問控制、與 GitHub Actions 的無縫集成以及支持存儲公共和私有image。