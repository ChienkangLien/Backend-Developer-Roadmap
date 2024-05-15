# Redis
* 鍵值（key-value）型，value 支持多種不同數據結構，功能豐富
* 單線程，每個命令具備原子性
* 低延遲，速度快（基於內存、IO 多路復用、良好的編碼）
* 支持數據持久化（定期將內存搬運到磁盤）
* 支持主從集群、分片集群（數據拆分）
* 支持多語言客戶端

## 安裝
以CentOS 7 示範，這裡使用Redis 6.2.14
1. 安裝Redis 依賴`yum install -y gcc tcl`
2. 將redis-6.2.14.tar.gz 上傳到虛擬機的任意目錄，這裡放到/usr/local/src
3. 解壓`tar -zxvf redis-6.2.14.tar.gz`
4. `cd redis-6.2.14/`、`make && make install` (默認安裝目錄在/usr/local/bin)

### 啟動方式
#### 默認啟動
任意目錄執行`redis-server`，需要一直掛著頁面
#### 指定配置啟動
以後台方式啟動，修改redis.conf
1. `cd /usr/local/src/redis-6.2.14/`
2. `cp redis.conf redis.conf.bak` 備份原始檔案
3. `vi redis.conf`
```
#監聽的地址，默認是只能在本地訪問，修改為0.0.0.0則可以再任意IP訪問，正式環境不要這樣設
#bind 127.0.0.1 -::1
bind 0.0.0.0

#守護進程，yes可在後台運行
#daemonize no
daemonize yes

#密碼，設置後訪問redis必須輸入密碼
requirepass password

### 以下說明其他常用的配置
#監聽端口
port 6379

#工作目錄(默認當前目錄)，也就是運行redis-server時的命令，日誌、持久化等文件會保存在這下
dir ./

#資料庫數量(默認16個，編號0-15)
databases 16

#最大內存
# maxmemory <bytes>

#日誌文件，默認不紀錄
#logfile ""
logfile "redis.log"
```
4. `cd /usr/local/src/redis-6.2.14`、`redis-server redis.conf` 啟動
5. `ps -ef | grep redis` 驗證，會得到redis-server 0.0.0.0:6379
6. `kill -9 進程號` 關閉(強制但是不安全)

#### 開機自動啟動
1. `vi /etc/systemd/system/redis.service` 新建文件
```
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /usr/local/src/redis-6.2.14/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
2. `systemctl daemon-reload` 重新加載系統服務
3. `systemctl start redis` 啟動
4. `systemctl status redis` 驗證
5. `systemctl stop redis` 關閉
6. `systemctl restart redis` 重啟
7. `systemctl enable redis` 設置開機自啟

## Redis 客戶端
### 命令行客户端
Redis 安裝完成後就自帶了命令行客戶端，`redis-cli [options] [commonds]`
其中常見的options有：
* `-h 127.0.0.1`：指定要連接的redis 節點的IP 地址，默認是127.0.0.1
* `-p 6379`：指定要連接的redis 節點的端口，默認是6379
* `-a 123321`：指定redis 的訪問密碼，若沒輸入密碼爾後要再透過`AUTH password` 輸入

commonds 就是Redis 的操作命令，例如：`ping`；不指定commond 時，會進入redis-cli 的交互控制台

### 圖形化桌面客戶端
https://github.com/lework/RedisDesktopManager-Windows
這裡下載2022.5 ，解壓後即可安裝
