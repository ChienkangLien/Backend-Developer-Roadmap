# Jenkins_Monolithic
此練習VM 統一使用CentOS7

| 名稱 | IP | 安裝軟體 |
| -------- | -------- | -------- |
| 程式託管  | Internet | Gitlab |
| 持續集成  | 192.168.191.132 | Jenkins、JDK17、Maven、Git、SonarQube |
| 程式運行  | 192.168.191.134 | JDK17、Tomcat9 |

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

## Tomcat
使用獨立Tomcat 運行
1. 安裝JDK
```shell=
cd /usr/lib/jvm
wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz
tar xf jdk-17_linux-x64_bin.tar.gz
vi /etc/profile
#最末添加以下內容
export JAVA_HOME=/usr/lib/jvm/jdk-17.0.10
export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
#退出編輯後
source /etc/profile
#驗證
java -version
```
2. 把Tomcat 壓縮檔上傳到VM 中並解壓：`tar -xzf apache-tomcat-9.0.86.tar.gz`
3. 創建目錄：`mkdir -p /opt/tomcat`
4. 移動檔案：`mv /root/apache-tomcat-9.0.86/* /opt/tomcat`
5. 新增用戶，為了讓Jenkins 所在的服務器使用；`vi /opt/tomcat/conf/tomcat-users.xml`
```xml=
<tomcat-users>
  <user username="devuser" password="password" roles="manager-gui,manager-script,tomcat,admin-gui.admin-script"/>
</tomcat-users>
```
6. 為了讓用戶的配置生效，還要修改context.xml：`vi /opt/tomcat/webapps/manager/META-INF/context.xml`，將以下註釋掉
```xml=
<!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
```
7. 啟動：`/opt/tomcat/bin/startup.sh`


## Jenkins
### 安裝
1. Jenkins 需要依賴JDK，因為是使用docker 安裝Jenkins，所以也是在container 中做安裝(若容器自帶則免安裝)
2. docker pull：`docker pull jenkins/jenkins`
3. docker run：`docker run -d -p 8080:8080 -p 50000:50000 --name jenkins jenkins/jenkins`
4. 訪問 http://192.168.191.132:8080/ 即可進一步安裝插件以及配置使用者資訊；這邊先安裝推薦插件以及跳過使用者配置(以admin 登入、默認密碼在容器中的`/var/jenkins_home/secrets/initialAdminPassword`)，這邊生成`edf5f58fdc0e47ac9b9f321a0b2248e6`

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
3. 建置，即會將代碼pull 下來

若是自建Gitlab 服務器，可改用公私鑰方式驗證

### 配置JDK 以及Maven
Tools 中配置這兩項的安裝目錄；若是在Jenkins 容器中，可透過echo $JAVA_HOME 以及mvn -v 來查看

接著在先前建立的FreeStyle 專案，即可在Configuration/Build Steps/執行 Shell 加入指令來做打包
```shell=
echo "開始編譯和打包"
mvn clean package
echo "編譯和打包結束"
```

### 遠程部屬Tomcat
以下採用先前創建的FreeStyle 軟體專案操作
1. 安裝插件Deploy to container
2. 配置Configuration/建置後動作
3. WAR/EAR files：target/*.war 選擇war 檔
4. Containers：Tomcat 8.x Remote 選擇對應版本
5. Credentials：以先前在Tomcat 新增的devuser/password 作為憑證
6. Tomcat URL：http://192.168.191.134:8080 輸入對應位址

### 構建Maven 專案
1. 安裝Maven Integration 插件
2. 配置步驟與先前FreeStyle 專案一樣，唯一不同是Maven 指令輸入在Goal 及選項中(clean package)、不用敲在Shell 欄位

### 構建Pipeline 專案
以代碼(Declarative(聲明式)和 Scripted Pipeline(腳本式)語法；基於groovy)的形式實現，有兩種創建方法：可以直接在Jenkins 的UI 中輸入腳本；也可以通過創建一個Jenkinsfile 腳本文件放入代碼庫中。

#### Declarative
```groovy=
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                echo 'pull code'
            }
        }
        stage('build project') {
            steps {
                echo 'build project'
            }
        }
        stage('publish project') {
            steps {
                echo 'publish project'
            }
        }
    }
}
```
* stages：代表整個流水線的所有執行階段。通常stages 只有1個，包含多個stage。
* stage：代表流水線中的某個階段，可能出現n個。一般分為拉取代碼，編譯構建，部署等階段。
* steps：代表一個階段內需要執行的邏輯。steps 裡面是shell 腳本，git 拉取代碼，ssh 遠程發布等任意內容。
#### Scripted Pipeline
```groovy=
node {
    def mvnHome
    stage('pull code') {
        echo 'pull code'
    }
    stage('build project') {
        echo 'build project'
    }
    stage('publich project') {
        echo 'publish project'
    }
}
```
* Node：節點，一個Node 就是一個Jenkins 節點，Master 或者 Agent 是執行Step 的具體運行環境，後續講到Jenkins 的Master-Slave 架構的時候用到。
* Stage：階段，一個Pipeline 可以劃分為多個Stage，每個Stage 代表一組操作，比如：Build、Test、Deploy，Stage 是一個邏輯分組的概念。
* Step：步驟，Step 是最基本的操作單元，可以是打印一句話，也可以是構建一個Docker Image，由各類Jenkins 插件提供，比如命令：sh 'make'，就相當於我們平時shell 終端中執行make 命令一樣。

可以使用當中的Pipeline Syntax/Snippet Generator 工具輔助腳本的生成，例如：
1. 拉取代碼：Sample Step：checkout: Check out from version control、指定Repository URL 和Branch
2. 打包程式：Sample Step：Shell Script、Shell Script：mvn clean package
3. 部屬專案：Sample Step：deploy: Deploy war/ear to a container，指定WAR/EAR files 和Credentials 和Tomcat URL

即可組成以下
```groovy=
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://gitlab.com/chienkang1114/demo.git']])
            }
        }
        stage('build project') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('publish project') {
            steps {
                deploy adapters: [tomcat8(credentialsId: '88b48a98-4d97-4807-9e15-1ff9de4accc8', path: '', url: 'http://192.168.191.134:8080')], contextPath: null, war: 'target/*.war'
            }
        }
    }
}
```

Configuration/Pipeline/Definition 有兩種維護腳本方式
* Pipeline script：直接在Jenkins 的UI 中輸入腳本
* Pipeline script from SCM：創建一個Jenkinsfile 腳本文件放入代碼庫中，選擇Git 並指定Repository URL 和Branch，而Script Path 默認為Jenkinsfile，所以文件要放在根目錄並使用默認的命名

#### 參數構建
Configuration/參數化建置、可以增加自定義的參數，例如：將以上Jenkinsfile 內容修改`checkout scmGit(branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[url: 'https://gitlab.com/chienkang1114/demo.git']])`，Jenkins 配置文字參數branch，即可帶參數建置

### Build Triggers
常用的4種構建觸發器：
1. 遠端觸發建置：驗證 Token 輸入mytoken，訪問JENKINS_URL/job/demo_pipeline/build?token=TOKEN_NAME 即可觸發
2. 在其他專案建置完成後建置：指定要監控的專案，在該專案建置後觸發
3. 定期建置：輸入定時表達式構建(字符串從左往右分別為：分 時 日 月 周)
4. 輪詢SCM：當代碼來源是SCM則可使用，也是輸入定時表達式，接著監測指定的
5. Build when a change is pushed to GitLab.：由GitLab 發送建構請求(默認Jenkins 即會對push、merge 做響應)，另一方面也要到GitLat 配置Webhooks(也可選擇哪些操作觸發請求)

### 郵件配置
這裡使用Gmail 為範例
1. 安裝Email Extension Template 插件(獲得更多郵件功能)
2. 開始System配置 Jenkins 位置/系統管理員郵件地址：可順便依照需求給定信箱
3. 擴充電子郵件通知 SMTP server：smtp.gmail.com、SMTP Port：465、Credentials：Username with password(需另到Google 申請應用程式密碼)、Use SSL、預設收件人：收件者信箱
4. 電子郵件通知 SMTP 伺服器：smtp.gmail.com、Use SMTP Authentication：輸入使用者名稱/密碼、使用 SSL、SMTP 連接埠：465
5. 即可寄測試信，看看電子郵件通知的設定正不正確

使用郵件模板寄信
1. System配置 擴充電子郵件通知/預設內容類型 設定HTML
2. 將模板放在GitLab 目標repository 跟目錄下(email.html)
3. 模板中的參數可以參考 擴充電子郵件通知/內容 Token 參考
```htmlmixed=
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次構建日志</title>
</head>

<body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4"
      offset="0">
<table width="95%" cellpadding="0" cellspacing="0"
       style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
    <tr>
        <td>(本郵件是程序自動下發的，請勿回覆！)</td>
    </tr>
    <tr>
        <td><h2>
            <font color="#0000FF">構建結果 - ${BUILD_STATUS}</font>
        </h2></td>
    </tr>
    <tr>
        <td><br />
            <b><font color="#0B610B">構建信息</font></b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td>
            <ul>
                <li>項目名稱&nbsp;：&nbsp;${PROJECT_NAME}</li>
                <li>構建編號&nbsp;：&nbsp;第${BUILD_NUMBER}次構建</li>
                <li>觸發原因：&nbsp;${CAUSE}</li>
                <li>構建日志：&nbsp;<a href="${BUILD_URL}console">${BUILD_URL}console</a></li>
                <li>構建&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${BUILD_URL}">${BUILD_URL}</a></li>
                <li>工作目錄&nbsp;：&nbsp;<a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a></li>
                <li>項目&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${PROJECT_URL}">${PROJECT_URL}</a></li>
            </ul>
        </td>
    </tr>
    <tr>
        <td><b><font color="#0B610B">Changes Since Last
            Successful Build:</font></b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td>
            <ul>
                <li>歷史變更記錄 : <a href="${PROJECT_URL}changes">${PROJECT_URL}changes</a></li>
            </ul> ${CHANGES_SINCE_LAST_SUCCESS,reverse=true, format="Changes for Build #%n:<br />%c<br />",showPaths=true,changesFormat="<pre>[%a]<br />%m</pre>",pathFormat="&nbsp;&nbsp;&nbsp;&nbsp;%p"}
        </td>
    </tr>
    <tr>
        <td><b>Failed Test Results</b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td><pre
                style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">$FAILED_TESTS</pre>
            <br /></td>
    </tr>
    <tr>
        <td><b><font color="#0B610B">構建日志 (最後 100行):</font></b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td><textarea cols="80" rows="30" readonly="readonly"
                      style="font-family: Courier New">${BUILD_LOG, maxLines=100}</textarea>
        </td>
    </tr>
</table>
</body>
</html>
```
4. 修改流水線腳本，可在專案配置中使用Pipeline Syntax 輔助；Declarative Directive Generator/Sample Directive：post: Post Stage or Build Conditions、Always run, regardless of build status；即可得到
```groovy=
post {
  always {
    // One or more steps need to be included within each condition's block.
  }
}
```
5. 一樣在Pipeline Syntax 中的Snippet Generator/Sample Step：emailext: Extended Email、To：收件者信箱、Subject：`構建通知：${PROJECT_NAME} - Build # ${BUILD_NUMBER} - ${BUILD_STATUS}!`、Body：`${FILE,path="email.html"}`；即可得到`emailext body: '${FILE,path="email.html"}', subject: '構建通知：${PROJECT_NAME} - Build # ${BUILD_NUMBER} - ${BUILD_STATUS}!', to: '收件者信箱'`
6. 將以上兩步的結果加入到Jenkinsfile，內容在stages 之後並與之同級
```groovy=
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout scmGit(branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[url: 'https://gitlab.com/chienkang1114/demo.git']])
            }
        }
        stage('build project') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('publish project') {
            steps {
                deploy adapters: [tomcat8(credentialsId: '88b48a98-4d97-4807-9e15-1ff9de4accc8', path: '', url: 'http://192.168.191.134:8080')], contextPath: null, war: 'target/*.war'
            }
        }
    }
    post {
        always {
            emailext body: '${FILE,path="email.html"}', subject: '構建通知：${PROJECT_NAME} - Build # ${BUILD_NUMBER} - ${BUILD_STATUS}!', to: 'chienkang1114@gmail.com'
        }
    }
}
```

### 整合SonarQube
(無法使用JDK8，SonarQube與Jenkins 無法整合，故選擇JDK17)
1. pull sonarqube image：`docker pull sonarqube`，7.9以後的版本，只能結合Oracle / Microsoft SQL Server / PostgreSQL
2. pull postgres image：`docker pull postgres`
3. 啟動postgres 容器(默認帳號為postgres)
```shell=
docker run \
    --name postgresql \
    -e POSTGRES_PASSWORD=password \
    -p 5432:5432 \
    -v /root/postgres/data:/var/lib/postgresql/data \
    -d postgres
```
4. 連接postgres(192.168.191.132:5432) 並創建database sonar
5. 啟動sonarqube 容器(默認連線URL 的schema 為public，若要更改可加上?currentSchema=my_schema)
```shell=
docker run -d --name sonarqube \
    -p 9000:9000 --link postgresql:db \
    -e SONAR_JDBC_URL=jdbc:postgresql://192.168.191.132:5432/sonar \
    -e SONAR_JDBC_USERNAME=postgres \
    -e SONAR_JDBC_PASSWORD=password \
    -v sonarqube_data:/opt/sonarqube/data \
    -v sonarqube_extensions:/opt/sonarqube/extensions \
    -v sonarqube_logs:/opt/sonarqube/logs \
    sonarqube
```
6. 訪問http://192.168.191.132:9000/ 並登入，默認帳密admin/admin，並生成token(提供給Jenkins連接)，這邊生成`sqa_f4c75f120fcf26a4189bcfeff98e07aebe93fa42` 並加入到Jenkins 的Credentials(Secret text)
7. 在Jenkins 安裝SonarQube Scanner 插件並透過此插件在VM 中安裝SonarQube Scanner，Tools/SonarQube Scanner 安裝、name：自訂(這裡叫做sonarqube-scanner)、自動安裝、版本：最新
8. 配置Sonar服務，System/SonarQube servers/Add SonarQube，name：自訂(這裡叫做sonarqube)、Server URL：http://192.168.191.132:9000/

接著先說明非流水線專案如何套用，以先前創建的FreeStyle 專案為例
1. Configuration/Build Steps/新增建置步驟/Execute SonarQube Scanner，JDK：jdk17、Analysis properties：輸入以下
```shell=
# must be unique in a given SonarQube instance
sonar.projectKey=web_demo_freestyle
# this is the name and version displayed in the SonarQube UI. Was mandatory prior to SonarQube 6.1.
sonar.projectName=web_demo_freestyle
sonar.projectVersion=1.0

# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
# This property is optional if sonar.modules is set.
sonar.sources=.
sonar.exclusions=**/test/**,**/target/**
sonar.java.binaries=target/classes

sonar.java.source=17
sonar.java.target=17

# Encoding of the source code. Default is default system encoding
sonar.sourceEncoding=UTF-8
```

接著說明流水線如何套用，以先前創建的Pipeline 專案為例
1. 將以下內容push 到Git 專案中(sonar-project.properties)，
```shell=
# must be unique in a given SonarQube instance
sonar.projectKey=web_demo_pipeline
# this is the name and version displayed in the SonarQube UI. Was mandatory prior to SonarQube 6.1.
sonar.projectName=web_demo_pipeline
sonar.projectVersion=1.0

# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
# This property is optional if sonar.modules is set.
sonar.sources=src/main/java
sonar.java.binaries=target/classes

sonar.java.source=17
sonar.java.target=17

# Encoding of the source code. Default is default system encoding
sonar.sourceEncoding=UTF-8
```
2. 修改Git 專案中的Jenkinsfile
```groovy=
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout scmGit(branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[url: 'https://gitlab.com/chienkang1114/demo.git']])
            }
        }
        stage('SonarQube checking') {
			steps{
				script {
					// 引入sonarqubescanner工具
					scannerHome = tool 'sonarqube-scanner'
				}
				//引入sonarqube的服務器
				withSonarQubeEnv('sonarqube') {
					sh "${scannerHome}/bin/sonar-scanner"
				}
			}
		}
        stage('build project') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('publish project') {
            steps {
                deploy adapters: [tomcat8(credentialsId: '88b48a98-4d97-4807-9e15-1ff9de4accc8', path: '', url: 'http://192.168.191.134:8080')], contextPath: null, war: 'target/*.war'
            }
        }
    }
    post {
        always {
            emailext body: '${FILE,path="email.html"}', subject: '構建通知：${PROJECT_NAME} - Build # ${BUILD_NUMBER} - ${BUILD_STATUS}!', to: 'chienkang1114@gmail.com'
        }
    }
}
```