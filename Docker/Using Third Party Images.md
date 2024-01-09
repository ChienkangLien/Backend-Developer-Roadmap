# Using Third Party Images
第三方image 是預先構建的 Docker 容器image，可在 Docker Hub(最大、最受歡迎的容器image 注冊表) 或其他容器注冊表中找到。這些image 由個人或組織創建和維護。

請記住，第三方image 可能存在安全漏洞或配置錯誤。在將其用於生產環境之前，請始終驗證image 的來源。

## Databases
### MySQL
`docker pull mysql` 拉取 MySQL image
`docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 -d mysql`
* docker run：用於創建並啟動一個容器。
* --name some-mysql：指定容器的名稱為 some-mysql。
* -e MYSQL_ROOT_PASSWORD=my-secret-pw：設置 MySQL root 用戶的密碼為 my-secret-pw。
* -p 3306:3306：將容器的 3306 端口映射到主機的 3306 端口。這樣，可以通過主機的 3306 端口訪問 MySQL 服務。第一個數字是主機的端口，第二個是容器的端口。
* -d mysql：指定要使用的 Docker 鏡像，這里使用的是官方的 MySQL 鏡像。-d 參數是在後台運行容器。

`docker exec -it my-mysql mysql -u root -p` 連接到運行中的 MySQL 容器

### PostgreSQL
`docker pull postgres` 拉取PostgreSQL image
`docker run --name some-postgres -e POSTGRES_PASSWORD=my-secret-pw -p 5432:5432 -d postgres`
* -e POSTGRES_PASSWORD=my-secret-pw：設置 PostgreSQL 用戶 postgres 的密碼為 my-secret-pw。這個環境變量設置了 PostgreSQL 的密碼，確保您設置一個安全的密碼。
* -p 5432:5432：將容器的 5432 端口映射到主機的 5432 端口。第一個數字是主機的端口，第二個是容器的端口。
* -d postgres：指定要使用的 Docker 鏡像，這里使用的是官方的 PostgreSQL 鏡像。-d 參數是在後台運行容器。

`docker exec -it some-postgres psql -U postgres` 連接到運行中的 PostgreSQL 容器

### MongoDB
`docker pull mongo`
`docker run --name some-mongo -p 27017:27017 -d mongo`
`docker exec -it some-mongo mongo`

## Interactive Test Environments with Docker
Docker允許您創建隔離的、一次性的環境，在測試完成後可以刪除。這使得與第三方軟件一起工作、測試不同的依賴庫或版本，以及快速進行實驗而無需擔心損壞本地設置變得更加容易。

* 要使用Python映像啟動交互式測試環境：

`docker run -it --rm python`
-it標誌確保您以tty的交互模式運行容器，--rm標誌將在停止時刪除容器。

現在您會在容器內部的交互式Python shell中。可以執行任何Python命令，或者使用pip安裝其他包。
完成交互式會話時，輸入exit()或按CTRL+D退出容器、容器也將自動刪除。

* Node.js：要啟動交互式Node.js shell：

`docker run -it --rm node`
* Ruby：要啟動交互式Ruby shell

`docker run -it --rm ruby`
* MySQL：要啟動臨時MySQL實例

`docker run -it --rm --name temp-mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -p 3306:3306 mysql`

## Command Line Utilities
Docker image可以包含命令行工具或獨立應用程序，我們可以在容器內運行這些工具。

* BusyBox
是一個小巧（1-2 Mb）且簡單的命令行應用程序，提供了許多常用的Unix 工具，如awk、grep、vi 等。要在Docker 容器內運行BusyBox，只需拉取映像並使用Docker運行它：
```
docker pull busybox
docker run -it busybox /bin/sh
```
進入容器後，便可像在常規命令行上一樣運行各種BusyBox工具。
* cURL
可以使用各種網絡協議進行數據傳輸。它通常用於測試API或從互聯網下載文件。
```
docker pull curlimages/curl
docker run --rm curlimages/curl https://example.com
```

其他命令行工具
Docker image中還有大量命令行工具，包括：

wget：用於從網絡非交互式下載文件的免費實用程序。
ImageMagick：用於圖像處理和轉換的強大軟件套件。
jq：輕量靈活的命令行JSON處理器。