# CI/CD
此練習VM 統一使用CentOS7

| 名稱 | IP | 安裝軟體 |
| -------- | -------- | -------- |
| 程式託管  | Internet | Gitlab |
| 持續集成  | 192.168.191.132 | Jenkins、JDK、Maven、Git、SonarQube |
| 程式運行  | 192.168.191.133 | JDK、Tomcat |

## GitLab
在VM 啟用gitlab 耗時過久、改用線上gitlab；仍提供安裝方法
### package 安裝
1. 安裝相關依賴：`yum install -y policycoreutils openssh-server openssh-clients postfix`
2. 啟動ssh服務、設置為開機啟動：`systemctl enable ssh && sudo systemctl start sshd`，驗證`systemctl status sshd`
3. 設置postfix 為開機啟動，postfix 支援gitlab 發信功能：`systemctl enable postfix && systemctl start postfix`
4. 開放ssh 以及http 服務，然後重新加載防火牆列表：
```shell=
firewall-cmd --permanent --add-service=ssh
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```
如果防火牆關閉、就不需要以上配置
5. 下載gitlab：`curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash`
6. 安裝：`sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-ee` 將URL 改成自己的domain，這裡使用`sudo EXTERNAL_URL="http://192.168.191.132:82" yum install -y gitlab-ee`
7. 如果防火牆沒關閉，將port 添加到防火牆(這裡使用82 port)：
```shell=
firewall-cmd --zone=public --add-port=82/tcp --permanent
firewall-cmd --reload
```
8. 啟動
```shell=
sudo gitlab-ctl reconfigure
sudo gitlab-ctl start
```
### docker 安裝
1. 創建docker-compose.yml(82 port、2222 port)
```yaml=
version: '3'
services:
  gitlab:
    image: gitlab/gitlab-ee:latest
    container_name: gitlab
    restart: always
    hostname: 192.168.191.132
    ports:
      - "82:80"
      - "443:443"
      - "2222:22"
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://192.168.191.132'
    volumes:
      - '/srv/gitlab/config:/etc/gitlab'
      - '/srv/gitlab/logs:/var/log/gitlab'
      - '/srv/gitlab/data:/var/opt/gitlab'
```
2. 啟用：`docker-compose up -d`

### 角色權限
* Owner（所有者）： 擁有項目的完全控制權，可以管理項目的成員、設置、刪除項目。Owner角色通常是項目的創建者。
* Master（主管理員）： 可以對項目進行管理操作，如添加成員、推送代碼等，但不能刪除項目或轉讓項目所有權。
* Developer（開發者）： 可以clone、push 代碼、創建分支、合並請求等操作，但不能管理項目成員。
* Reporter（報告者）： 可以clone 代碼，但不能提交。
* Guest（訪客）： 可以創建issue、發表評論，不能進行任何修改操作。

## Git
安裝`yum install git -y`；因為是使用docker 安裝Jenkins，所以也是在container 中做安裝(若容器自帶則免安裝)

## Maven
安裝：`yum install maven -y`；因為是使用docker 安裝Jenkins，所以也是在container 中做安裝(若容器自帶則免安裝)
```shell=
docker exec -u root -it jenkins /bin/bash

apt-get update
apt-get install -y maven
```

## Jenkins
### 安裝
1. Jenkins 需要依賴JDK：`sudo yum install java-1.8.0-openjdk-devel`；因為是使用docker 安裝Jenkins，所以也是在container 中做安裝(若容器自帶則免安裝)
2. docker pull：`docker pull jenkins/jenkins`
3. docker run：`docker run -d -p 8080:8080 -p 50000:50000 --name jenkins jenkins/jenkins`
4. 訪問 http://192.168.191.132:8080/ 即可進一步安裝插件以及配置使用者資訊；這邊先安裝推薦插件以及跳過使用者配置(以admin 登入、默認密碼在`/var/jenkins_home/secrets/initialAdminPassword`)

### 角色權限
1. 安裝Role-based Authorization Strategy 並在Security/授權 選擇Role-Based Strategy 開啟功能。
2. Manage and Assign Roles/Manage Roles 指定角色權限
3. Manage and Assign Roles/Assign Roles 指派角色

### 連接Gitlab
1. 在Gitlab 新增token：User Settings/Access Tokens；Scopes 選擇api 即可
2. 安裝Gitlab 插件
3. 配置System/GitLab，因為用的是GitLab 網站服務器、host URL填入https://gitlab.com

簡易測試
1. 建立FreeStyle 軟體專案
2. 原始碼管理選Git、輸入對應repository URL 以及分支；這裡不需再輸入憑證、因為先前連接Gitlab 已經有帶入權限
3. 建置

若是自建Gitlab 服務器，可改用公私鑰方式驗證

### 配置JDK 以及Maven
Tools 中配置這兩項的安裝目錄；若是在Jenkins 容器中，可透過echo $JAVA_HOME 以及mvn --version 來查看

接著在先前建立的FreeStyle 專案，即可在Configuration/Build Steps 加入`mvn clean package` 來做打包