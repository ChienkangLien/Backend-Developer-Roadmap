# Nginx
## 環境設定
練習用
1. `systemctl stop firewalld` 關閉防火牆(重開機的話會恢復enable，或是直接改用disable)
2. `vim /etc/selinux/config` 停用elinux(一種Linux內核安全模塊)，須重啟
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
#SELINUX=enforcing
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```
## 安裝
https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/
### 源碼安裝
省略參數的配置
1. ` yum install -y gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel`
* gcc：GCC 編譯器，因為Nginx 透過C 語言編寫，需要一個編譯器來處理
* pcre：兼容正則表達式庫，Nginx 的Rewrite 模塊和http 核心模塊會使用
* zlib：壓縮算法
* openssl：進行安全通信，如果服務器需要提供安全網頁時就需要使用

2. `gcc --version`
`rpm -qa pcre pcre-devel zlib zlib-devel openssl openssl-devel` 檢查是否安裝成功
3. `wget https://nginx.org/download/nginx-1.24.0.tar.gz` 到官網確定stable 版本
4. `mkdir -p nginx/core` 創建資料夾方便管理(可選)
`mv nginx-1.24.0.tar.gz nginx/core/` (可選)
5. `cd nginx/core`
`tar -zxf nginx-1.24.0.tar.gz` 解壓縮
6. `cd nginx-1.24.0/`
`./configure` 安裝；可在這個指令後添加需要的參數，若無手動加參數則參數為空，這次練習用以下指令安裝
```
./configure --prefix=/usr/local/nginx \
--sbin-path=/usr/local/nginx/sbin/nginx \
--modules-path=/usr/local/nginx/modules \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--error-log-path=/usr/local/nginx/logs/error.log \
--http-log-path=/usr/local/nginx/logs/access.log \
--pid-path=/usr/local/nginx/logs/nginx.pid \
--lock-path=/usr/local/nginx/logs/nginx.lock
```
7. `make && make install` 編譯
這個安裝動作沒有指定安裝目錄，默認會安裝在/usr/local/nginx
8. `cd /usr/local/nginx/sbin/`
`./nginx` 執行
在瀏覽器訪問VM 的IP位置(無須指定port)，即可看到歡迎頁面
`./nginx -V` 可查看參數(configure arguments)
### yum 安裝
參數會有默認配置
1. `yum install yum-utils` 工具包
2. `vi /etc/yum.repos.d/nginx.repo` 貼上以下文件
```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```
3. `yum install -y nginx` 安裝
4. `whereis nginx`
`cd /usr/sbin`
`./nginx` 執行
在瀏覽器訪問VM 的IP位置(無須指定port)，即可看到歡迎頁面
`./nginx -V` 可查看參數(configure arguments)

檢視目錄結構
`yum install -y tree` 安裝樹狀檢視工具
`tree /usr/local/nginx` 也就是 --prefix 參數的值

### 配置系統服務
1. `vim /usr/lib/systemd/system/nginx.service`
```
[Unit]
Description=nginx web service
Documentation-http://nginx.org/en/docs
After=network.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/comf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
PrivateTmp=true

[Install]
WantedBy=default.target
```
2. `chmod 755 /usr/lib/systemd/system/nginx.service`

可以透過systemctl start|restart|status|stop|reload|enable nginx 來控制
### 配置到系統環境
1. `vim /etc/profile`
2. 在最後一行加上`export PATH=$PATH:/usr/local/nginx/sbin`
3. `source /etc/profile`


## 相關操作
`ps -ef | grep nginx` 檢視當前進程，以下帶入主進程(master process) pid
`kill -TERM {pid}` 立即關閉
`kill -QUIT {pid}` 優雅關閉
`kill -HUP {pid}` 重讀配置文件並啟用
`kill -USR1 {pid}` 重新打開日誌文件
`kill -USR2 {pid}` 升級服務使用
`kill -WINCH {pid}` 關閉子進程(worker process)

在sbin目錄底下
`./nginx -?` `./nginx -h` 檢視細節
`./nginx -t` `./nginx -T` 測試配置文件
`./nginx -s` 後面可接信號stop(快速關閉)、quit(優雅關閉)、reopen(類似USR1)、reload(類似HUP)

### 信號升級
新版本先配置和編譯、不需安裝，`./configure` 和`make` 就好
1. `cd usr/local/nginx/sbin`
`mv nginx nginxold` 當前版本備份
2. `cd ~/nginx/core/{新版本}/objs`
`cp nginx /usr/local/nginx/sbin` 將新版本編譯後的objs目錄下的nginx文件複製到原來的/usr/local/nginx/bin目錄
3. `kill -USR2 {pid}` 發送USR2信號給舊版本的pid
4. `kill -QUIT {oldpid}` 發送QUIT信號給舊版本的pid

### make 升級
新版本先配置和編譯、不需安裝，`./configure` 和`make` 就好
1. `cd usr/local/nginx/sbin`
`mv nginx nginxold` 當前版本備份
2. `cd ~/nginx/core/{新版本}/objs`
`cp nginx /usr/local/nginx/sbin` 將新版本編譯後的objs目錄下的nginx文件複製到原來的/usr/local/nginx/bin目錄
3. `cd ..`
`make upgrade` 在~/nginx/core/{新版本} 目錄底下升級

## 配置文件
`cat /usr/local/nginx/conf/nginx.conf`
基本配置(刪掉#註解內容)
```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
### 全局
* `user username[usergroup]` 設置用戶[用戶組]
* `master_process on|off` 是否開啟工作進程，默認on
* `worker_processes num` 生成工作進程的數量，默認1，建議設置與CPU內核數一致
* `daemon on|off` 守護進程，默認on
* `pid file` pid存儲的文件路徑，默認/usr/local/nginx/logs/nginx.pid
* `error_log  file[日誌級別]` 日誌存放路徑
* `include file` 引用其他文件配置

### event
* `accept_mutex on|off` 網路連接序列化，默認on；解決驚群問題、一個請求驚動所有worker 進程爭奪，on 的話就只會喚醒一個worker 進程
* `multi_accept on|off` 允許同時接收多個網路連接，默認off，建議on
* `worker_connections num` worker 進程最大連接數，不能大於操作系統支持的max 數量
* `use method` 選擇哪種事件驅動來處理網路消息，默認依操作系統而定；select/poll/epoll/kqueue，建議epoll

### http
* `default_type  mime-type` 前端回應的MIME TYPE，默認text/plan
* `access_log  file[format[buffer=size]]` 訪問日誌，默認access_log  logs/access.log combined
* `log_format name [escape=default|json|none] string...` 輸出格式，默認log_format combined "..."
* `sendfile on|off` 使用sendfile()傳輸文件，默認off
* `keepalive_timeout time` 長連接時間，默認keepalive_timeout 75s
* `keepalive_requests num` keep-alive連接使用的次數，默認keepalive_requests 100
* `tcp_nopush on|off` 必須在sendfile打開時才會生效，提升網路包的傳輸效率，默認off
* `tcp_nodelay on|off` 必須在keep-alive連接打開時才會生效，提升網路包傳輸的實時性，默認on

sendfile 用於控制文件傳輸的機制。它允許Nginx 直接從磁盤讀取文件並發送到客戶端，而無需先將文件內容讀入用戶空間緩沖區。這種直接傳輸可以提高性能，減少CPU 和內存的使用，特別是在處理大文件時。
#### server
* `listen address[:port][default_server]...`
`listen port[default_server]` 監聽，默認listen *:80|*:8000；default_server屬性是標示符，用來配置默認主機，如果沒有匹配到對應的address:port，則會默認執行。如果不指定則默認使用的是第一個server
* `server_name name...` 設置主機名稱(精準、通配、正則表達)，默認server_name ""；因為域名是要收費的，可以使用修改hosts文件來製作虛擬域名(`/etc/hosts`)
* `location [=|~|~*|^~|@] uri{...}` 設置請求的URI
不帶符號，意味著以...開頭都可匹配，如/abc、則/abcef可匹配
=意味著精準匹配，?後接參數不影響仍可匹配
~意味著當前...包含正則表達，區分大小寫
~*意味著當前...包含正則表達，不區分大小寫
^~和不帶符號功能一樣，不同的是如果匹配了就停止搜尋其他模式
@搭配error_page使用
```
server{
    error_page 404 @jump_to_error;
    location @jump_to_error{
        default_type text/plain;
        return 404 'Not Found Page'
    }
}
```
* `root path` 請求的根目錄，默認 root html；處理結果：root 路徑+location路徑
* `alias path` 更改location 的URI；處理結果\：使用alias 路徑替換location 路徑(如果location 以/結尾、alias 也需/結尾)
* `index file...` 設置網站的默認首頁，默認index index.html
* `error_page code... [=[response]] uri` 設置網站的錯誤頁面

#### gzip
gzip 指令來自ngx_http_gzip_module 模塊，在nginx 安裝時內置到nginx 的安裝環境中
* `gzip on|off` 啟用gzip，默認gzip off
* `gzip_types mime-type ...` 根據回應的MIME TYPE 選擇性地開啟gzip 壓縮功能，默認gzip_types text/html
* `gzip_comp_level level` 設置壓縮程度，級別由1-9，壓縮程度高相對的效率低，默認gzip_comp_level 1
* `gzip_vary on|off` 是否攜帶 "Vary: Accept-Encoding" 的回應頭，用意是告訴接收方發送數據經過了gzip壓縮，默認gzip_vary off
* `gzip_buffers number size` 處理請求壓縮的緩衝區數量及大小，默認gzip_buffers 32 4k|16 8k
* `gzip_disable regex ...` 選擇性開啟gzip功能，根據瀏覽器標誌(user-agent)來做匹配
* `gzip_http_version 1.0|1.1` 根據http 版本選擇性開啟gzip，默認gzip_http_version 1.1
* `gzip_min_length length` 針對傳輸數據(根據Content-Length)的大小，選擇性開啟gzip，默認gzip_min_length 20(默認單位是bytes)
* `gzip_proxied off |expired|no-cache|no-store|private|no_last_modified|no_etag|auth|any;` 是否對服務端返回的結果進行壓縮，默認gzip_proxied off
off - 關閉
expired - 啟用，若有header: Expires
no-cache - 啟用，若有header: Cache-Control:no-cache
no-store - 啟用，若有header: Cache-Control:no-store
private - 啟用，若有header: Cache-Control:private
no_last_modified - 啟用，若沒有header: Last-Modified
no_etag - 啟用，若沒有header: ETag
auth - 啟用，若有header: Authorization
any - 啟用

來自ngx_http_gzip_static_module 模塊
* `gzip_static on|off|always` 檢查是否存在與請求文件對應的預壓縮（.gz 格式）的靜態文件，並在可能的情況下直接返回該預壓縮文件，而不是實時壓縮原始文件，默認gzip_static off

添加ngx_http_gzip_static_module 模塊
1. `cd /usr/local/nginx/sbin`
2. `./nginx -V` 複製原有的參數
3. `mv nginx nginxold` (若是升級的動作)
4. `cd /root/nginx/core/nginx-1.24.0`
5. `make clean` (若是升級的動作)清空之前編譯的內容
6. `./configure {第二步複製的參數} --with-http_gzip_static_module` 
7. `make` (若是升級的動作)
8. `cp objs/nginx /usr/local/nginx/sbin` (若是升級的動作)
9. `make upgrade` (若是升級的動作)

#### 緩存
* `expires [modified] time|epoch|max|off` 控制頁面緩存作用，默認expires off
[modified] time - 可以整數(Cache-Control為max-age=time)以及負數(Cache-Control為no-cache)，指定過期時間
epoch - Expires為1980-01-01 00:00:00、Cache-Control為no-cache
max - Expires為2037-12-31 23:59:59、Cache-Control為10年
off - 不緩存

* `add_header name value [always]` 添加指定的響應頭和值，ex: add_header Cache-Control no-store

## 安全性
### 跨域
在目標的location 中添加：
```
add_header Access-Control-Allow-Origin {目標網域}|*;
add_header Access-Control-Allow-Method GET,POST,PUT,DELETE;
```
### 防盜鏈
防止其他網站或第三方直接使用您網站的靜態資源
在目標的location 中添加：
```
valid_referers none|blocked|server_name|string...
if ($invalid_referer) {
    return 403; # 禁止訪問
}
```
none - 如果沒有Referer，允許訪問
blocked - 如果Referer不為空以及不為空白，允許訪問
server_name - 指定具體域名或IP
string - 支持正則表達或*

## rewrite
重定向 URL 或修改请求的 URI(在目標的location 中添加)；依賴PCRE的支持，因此在編譯安裝Nginx 前，需要安裝PCRE 庫；使用ngx_http_rewrite_module 來解析

rewrite常用全局變量：
* `$arg` URL中的參數
* `$http_user_agent` 用戶訪問服務的代理訊息
* `$host` 訪問服務器的server_name
* `$document_uri` 當前訪問地址URI
* `$document_root` 當前請求對應location 的root 值
* `$content_length` Content-Length 的值 
* `$content_type` Content-Type 的值

還有獲取客戶端訊息、請求訊息等等，可在 https://nginx.org/en/docs/varindex.html 查找

相關指令：
* `set #variable value` 設置變量
* `if (condition){...}` 條件判斷，if 後要有一個空格
* `break` 中斷當前作用域的指令，以及終止當前的匹配並把當前的URI在本location 進行重定向訪問
* `return [code] URL` `return code [text]` 向客戶端返回
* `rewrite regex replacement [flag]`
regex - 用來匹配URI的正則表達
replacement - 匹配成功後，用於替換URI 中被截取內容的字符串；如果該字符串是http:// 或 https:// 開頭、直接返回而不會繼續向下對URI 進行其他處理
```
location /old {
    rewrite ^/old(.*)$ /new$1;
}
```
將 /old 開頭的 URL 重寫為 /new。^/old(.*)$ 是一個正則表達式，匹配以 /old 開頭的 URL，並將其重寫為 /new。$1 表示正則表達式中括號捕獲到的內容

flag：
1. last：停止處理當前的 rewrite 指令集，並重新啟動匹配的位置。
2. break：停止處理當前的 rewrite 指令集，並不再對後續的 rewrite 指令進行匹配。
3. permanent：執行 301 永久重定向。
4. redirect：執行 302 臨時重定向。

* `rewrite_log on|off` URL重寫的日誌輸出，默認off；以notice 級別輸出到error.log
* `server_name_in_redirect on|off` 使用配置文件中的 server_name 指令中定義的值來構建重定向的URL，off 時使用客戶端請求的 Host 頭（即客戶端實際訪問的域名）來構建；默認off (0.8.48版之後)

## 代理
### 正向
配置在代理服務器上：
```
# 假設代理服務器的IP: 192.0.0.2
server {
    listen {服務器的port}; #ex: 82;
    resolver 8.8.8.8; #指定了 Nginx 在需要 DNS 解析時使用的 DNS 服務器。ex: 使用 Google Public DNS（8.8.8.8），用來解析proxy_pass中的域名
    location / {
        proxy_pass http://$host{:port}$request_uri;
    }
}
```
本機配置：網際網路選項 > 連線 > LAN設定 > Proxy伺服器 > 配置代理服務器的位址

### 反向
由ngx_http_proxy_module 模塊解析，在安裝時已經有加裝，配置在代理服務器上：
* `proxy_pass URL` 配置被代理服務器位址，可以是主機名、IP地址加埠號形式
```
server {
    listen 80;
    server_name localhost
    location /server{
        proxy_pass http://192.0.0.2/;
    }
}
#訪問http://{服務器位址}/server/index.html
#如果proxy_pass 結尾不加"/" ，會建構成http://localhost/server/index.html
#如果proxy_pass 結尾加了"/" ，會建構成http://localhost/index.html
```
* `proxy_set_header field value` 更改服務器收到的requesr header，然後將新的header發送到代理的服務器，默認proxy_set_header Host $proxy_host; proxy_set_header Connection close;
* `proxy_redirect default|off|redirect replacement` 重置herder Location 和Refresh 的值，默認proxy_redirect default
default - 將location 塊的uri 變量作為replacement，將proxy_pass 變量作為redirect 進行替換
redirect replacement - redirect: 目標，Location 的值replacement: 要替換的值
```
server {
    listen 8081;
    server_name localhost;
    location / {
        proxy_pass http://192.0.0.20:8082/;
        proxy_redirect http://192.0.0.20/ http://192.0.0.10/;
    }
}

server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://192.0.0.20/;
    }
}
```
訪問'http://192.0.0.10:8081/' 時，代理服務器會將Location 的值替換成'http://192.0.0.10/' ，這時還需另外配置80的跳轉到被代理的位址

* `proxy_buffering on|off` 代理服務器的緩衝區，默認on
* `proxy_buffers number size` 指訂單個連接從代理服務器讀取響應的緩存區的個數及大小，默認8 4K|8K(與系統平台有關)
* `proxy_buffer_size size` 設置從被代理服務器獲取的第一部分響應數據的大小，保持與proxy_buffers 中的size 一致即可，默認4k|8k(與系統平台有關)
* `proxy_busy_buffers_size size` 限制同時處於BUSY狀態的緩衝總大小，默認8k|16k(與系統平台有關)
* `proxy_temp_path path` 當緩衝區存滿後，仍未被Nginx 服務器完全接受，響應數據就會被臨時存放在硬碟文件上，配置該文件路徑，默認proxy_temp_path proxy_temp (path最多三層)
* `proxy_temp_file_write_size size` 設置硬碟上緩衝文件的大小，默認8k|16k(與系統平台有關)

## SSL
需要添加 --with-http_ssl_module，而該模塊在編譯過程中又需要OpenSSL的支持

相關指令，也是透過該模塊解析：
* `ssl on|off` 開啟HTTPS，更常見的啟用方式是在server塊中使用listen 443 ssl，默認off
* `ssl_certificate file` 為當前主機指定一個帶有PEM格式的證書
* `ssl_certificate_key file` 指定PEM secret key文件的路徑
* `ssl_session_cache off|none|[builtin[:size]][shared:name:size]` 配置用於SSL會話的緩存，默認none
off - 禁用，客戶端不得重複使用會話
none - 進用，客戶端可以重複使用會話，但是並沒有在緩存中存儲會話參數
builtin - 內置OpenSSL緩存，僅在一個工作進程中使用
shared - 所有工作進程共享緩存，緩存的相關信息用name 和size 來指定
* `ssl_session_timeout time` 開啟SSL會話功能後，客戶端能夠使用緩存中的會話參數的時間，默認5m
* `ssl_ciphers ciphers` 指出允許的密碼，密碼指定為OpenSSL支持的格式，默認HIGH:!aNULL:!MD5；可透過`opsenssl ciphers`查看允許的值
* `ssl_prefer_server_ciphers on|off` 指定是否服務器密碼優先客戶端密碼，默認off

### 生成證書
* 向第三方服務商購買(正式時使用)
關鍵的檔案為.key 和.pem
* 使用openssl 生成(測試時使用)
`mkdir /root/cert`
`cd /root/cert`
`openssl genrsa -des3 -out server.key 1024` 生成.key，這一步會要求輸入加密字段
`openssl req -new -key server.key -out server.csr` 生成.csr，這一步會要求輸入先前的加密字段以及相關個人信息
`cp server.key server.key.org`
`openssl rsa -in server.key.org -out server.key`
`openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt`
關鍵的檔案為.key 和.crt

example(這段配置可在/root/nginx/core/nginx-1.24.0/conf/nginx.conf 找到)，配置在:
```
server {
    listen       443 ssl;
    server_name  localhost;

    ssl_certificate      /root/cert/server.crt;
    ssl_certificate_key  /root/cert/server.key;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
         root   html;
        index  index.html index.htm;
    }
}
```

加碼，瀏覽器訪問http:// 自動加上s
```
server {
    listen 80;
    server_name example.com;
    location / {
        rewrite ^(.*) https://example.com$1;
    }
}
```

## 負載均衡
最簡單的方式可透過DNS輪詢，DNS 服務器會將同一個域名解析為多個 IP 地址列表(A 紀錄)，然後按順序依次返回這些 IP 地址給請求者。

Nginx 可以作為七層和四層負載均衡器，根據 OSI 模型中的層級來區分，它們有著不同的工作方式和用途。
* 七層負載均衡（Layer 7 Load Balancing）：
工作在應用層（Application Layer），能夠解析並理解 HTTP、HTTPS 等應用層協議。它可以根據請求的內容進行更加精細的負載均衡和流量管理，例如根據 URL、Cookie、HTTP 頭等信息進行負載均衡或路由。

主要特點：
基於應用層協議的信息進行負載均衡和流量分發。
能夠實現更精細的請求路由、內容轉發和會話保持等功能。
適用於需要更高級別的請求處理和路由的場景，如 Web 應用服務器的負載均衡。

* 四層負載均衡（Layer 4 Load Balancing）：
工作在傳輸層（Transport Layer），主要基於 IP 地址和端口信息進行負載均衡，它不會解析 HTTP 協議。它能夠實現對 TCP 和 UDP 等傳輸層協議的負載均衡。可以透過硬體以及軟體(LVS、HAProxy、Nginx)實現。

主要特點：
通過網絡層信息（IP 地址和端口）進行負載均衡。
只能實現基本的負載均衡，對請求內容不做深層次的解析或處理。
適用於高性能和低延遲要求的場景，如 TCP 流量負載均衡。

### 七層
需要用到proxy_pass 代理模塊，可以說是在反向代理基礎上把用戶的請求根據指定的算法分發到一組upstream 虛擬服務池。

* `upstream name {...}` 定義一組服務器，他們可以是監聽不同端口的服務器，也可以是同時監聽TCP 和Unix socket 的服務器，服務器可以指定不同的權重(默認為1)；配置在http 塊
* `server name [parameters]` 指定後端服務器的名稱和參數，可以是域名、IP、端口或是unix socket，配置在以上upstream 中
關於後端服務器的狀態配置(可選)
down - 標記該服務器永久不可用，不參與負載均衡
backup - 標記為備份服務器，當主服務器不可用時，才會參與
max_fails=num - 允許請求代理服務器失敗的次數，默認為1
fail_timeout=time - 經過max_fails 失敗後，服務暫停的時間，默認為10(秒)
max_conns=num - 設置代理服務器同時活動連接的最大數量，默認為0(不限制)

example:
```
upstream backend {
    server 192.0.0.10:8080;
    server 192.0.0.20:8080;
    server 192.0.0.30:8080;
}

server {
    listen 8001;
    server_name localhost;
    location / {
        proxy_pass http://backend;
    }
}
```
upstream 支持的分配算法：
輪詢 - 默認
weight=num - 權重(越大分配機率越大)，默認為1；寫在 server_name 之後
ip_hash - 依據IP分配；寫在upstream 塊中
least_conn - 依據最少連接；寫在upstream 塊中
url_hash - 依據URL分配；寫在upstream 塊中，寫的是hash &request_uri
fair - 依據響應時間；寫在upstream 塊中；默認不支持這個算法，需要加裝第三方模塊nginx-upstream-fair

下載模塊nginx-upstream-fair
1. 到 https://github.com/gnosek/nginx-upstream-fair 下載zip檔，上傳到指定目錄(/root/nginx/module)
2. `cd /root/nginx/module`
3. `unzip nginx-upstream-fair-master.zip`
4. `cd /root/nginx/core/nginx-1.24.0/`
5. `./configure {Nginx的安裝參數} --add-module=/root/nginx/module/nginx-upstream-fair-master`
6. `vim src/http/ngx_http_upstream.h`
7. 在struct ngx_http_upstream_srv_conf_s 塊中加入`in_port_t default_port;`
8. `make`
9. `mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginxold`
10. `cp objs/nginx /usr/local/nginx/sbin/`
11. `make upgrade`

### 四層
需要使用stream 模塊，編譯時加上--with-stream

* `stream {...}` 指定流服務器指令的配置文件上下文，和http 指令同級
* `upstream name {...}`

example(四層優先匹配於七層):
```
stream {
    upstream redisbackend {
        server 192.168.200.146:6379;
        server 192.168.200.146:6378;
    }
    upstream tomcatbackend {
        server 192.168.200.146:8080;
    }
    server {
        listen 81;
        proxy_pass redisbackend;
    }
    server {
        listen 82;
        proxy_pass tomcatbackend;
    }
}
```

## 緩存
0.7.48版開始提供緩存功能，使用ngx_http_proxy_module 模塊來完成
原理：
1. URL + [可指定其他內容] = key
2. key + MD5加密 = 字符串(密文)
3. 緩存目錄，ex: /usr/local/proxy_cache
4. 搭配字符串，在緩存目錄中分層存儲，ex: /usr/local/proxy_cache/a/b3
5. 判斷緩存目錄有沒有訪問數據對應的目錄

* `proxy_cache_path path [levels=number] keys_zone=zone_name:zone_size [inactive=time][max_sixe=size];` 設置緩存文件的存放路徑；配置在http 塊中
path - 緩存路徑，ex: /usr/local/proxy_cache
levels - 指定緩存空間對應的目錄，最多可以設置三層，每層取值為1|2，如：levels=1:2 兩層目錄，第一次是一個字母、第二次是兩個字母；舉例：example[key] 通過加密後值為1a79a4d60de6718e8e5b326e338ae533，levels=1:2 路徑為/usr/local/proxy_cache/3/53、levels=2:2:2 路徑為/usr/local/proxy_cache/33/e5/8a
keys_zone - 設置緩存區名稱及大小，如：keys_zone=:test:200m 緩存區名稱test、大小為200M，1M 大概可存儲8000個keys
inactive - 指定緩存的數據多久時間未被訪問就會被刪除，如：inactive=1d 一天內沒被訪問即刪除
max_size - 最大緩存空間，若空間存滿、默認會覆蓋最舊的資源，如：max_size=20g

example:
```
http{
    proxy_cache_path /usr/local/proxy_cache levels=1:2:1 keys_zone=test:200m inactive=1d max_size=20g;
}
```

* `proxy_cache zone_name on|off` 開啟代理緩存默認off
* `proxy_cache_key key` 設置web 緩存的key 值(會將key 值經過MD5 加密後緩存)，默認proxy_cache_key \$scheme\$proxy_host$request_uri
* `proxy_cache_valid [code...] time` 對不同返回狀態碼的URL 設置不同的緩存時間
* `proxy_cache_min_uses number` 資源被訪問多少次後被緩存
* `proxy_cache_methods GET|HEAD|POST` 緩存哪些HTTP 方法

example:
```
http{
    proxy_cache_path /usr/local/proxy_cache levels=1:2:1 keys_zone=test:200m inactive=1d max_size=20g;
    upstream backend {
        server 192.168.200.148:8080;
    }
    server {
        listen         8080;
        server_name    localhost;
        location / {
            proxy_cache test;
            proxy_cache_key mykey|$scheme$proxy_host$request_uri;
            proxy_cache_valid 200 5d;
            add_header nginx-cache "$upstream_cache_status"; #可選，值為HIT|MISS，意味是否命中緩存
            proxy_pass http://backend/js/;
        }
    }
}
```

緩存清除方法：
1. `rm -rf /usr/local/proxy_cache/...`
2. 使用第三方模塊ngx_cache_purge，安裝方式參考反向代理nginx-upstream-fair
需再配置文件加上：
```
#在同級的location 塊中加上
location ~ /purge(/.*) {
    proxy_cache_purge zone_name key
}
```
即可向 http://example.com/purge/URL 發送一個 HTTP 請求來執行清除操作，需注意設置的key 不得為\$scheme\$proxy_host$request_uri(URI會跑掉)

* `proxy_no_cache string ...` 定義不將數據進行緩存的條件
* `proxy_cache_bypass string ...` 定義不存緩存中獲取數據的條件
以上的條件皆建議配置為 $cookie_nocache $arg_nocache $arg_comment
$cookie_nocache - 當前請求中的 Cookie 是否包含了名為 nocache 的值。如果請求中包含名為 nocache 的 Cookie，則變量的值為 nocache，否則為一個空字符串。
$arg_nocache / $arg_comment - 當前請求的查詢參數中是否包含了名為 nocache / comment 的參數。如果請求的查詢參數中包含了名為 nocache / comment 的參數，則變量的值為 nocache / comment，否則為一個空字符串。

## 前後端分離
簡單範例，將靜態資源放在Nginx 安裝目錄中的html/web 底下
```
upstream webservice {
    server 192.168.200.146:8080;
}
server {
    listen    80;
    server_name localhost;
    # 動態資源
    location /demo {
        proxy_pass http://webservice;
    }
    # 靜態資源
    location ~/.*\.(png|jpg|gif|js) {
        root    html/web;
    }
    location / {
        root    html;
        index   index.html index htm;
    }
}
```
```htmlmixed=
<!DOCTYPE html>
<html>
<head>
    <title>Title</title>
    <script src="js/jquery.min.js"></script>
    <script>
        $(function(){
            // 或是'demo/getText'
            $.get('http://192.168.200.133/demo/getText',function(data){
                $("#msg").html(data);
            });
        });
    </script>
</head>
<body>
    <img src="images/logo.jpg"/>
    <p id="msg"></p>
</body>
</html>
```

## 高可用
本練習透過Keepalived 軟體達成，此軟體由C 編寫，最初是專為LVS 設計，通過VRRP 協議實現高可用功能。
### VRRP 協議
VRRP（Virtual Router Redundancy Protocol，虛擬路由器冗余協議）是一種網絡協議，用於提高路由器可靠性和冗余性。它允許多個路由器（實際上是虛擬路由器）共享一個虛擬 IP 地址，其中一個路由器處於活躍狀態並處理數據流量，而其他路由器處於備用狀態，當活躍路由器不可用時，備用路由器可以快速接管虛擬 IP 地址，確保網絡的連通性。

* 虛擬路由器： VRRP 將一組路由器配置為單個虛擬路由器，它們共享一個虛擬 IP 地址作為默認網關，提供給網絡中的設備。
* 選舉機制： 在 VRRP 組內，路由器通過選舉確定活躍路由器。優先級最高的路由器將成為活躍路由器，負責接收和處理數據包，其他路由器處於備用狀態。
* VRRP通告： 活躍路由器周期性地發送 VRRP 通告消息來表明自己的可用性。如果其他路由器停止接收這些通告，它們將推舉一台備用路由器成為新的活躍路由器。
* 快速故障轉移： 當活躍路由器不可用時（例如發生故障），VRRP 能夠快速地將虛擬 IP 地址轉移到備用路由器上，幾乎不會導致網絡中斷。
* 虛擬路由器MAC地址： 在網絡中，虛擬路由器有一個固定的虛擬 MAC 地址，當活躍路由器發生變化時，這個虛擬 MAC 地址也會切換到新的活躍路由器。

### 環境準備
| VIP | IP | 主機名 | 主/從 |
| -------- | -------- | -------- | -------- |
|  | 192.168.200.133  | keepalived1 | Master |
| 192.168.200.222 |  |  |  |
|  | 192.168.200.122  | keepalived2 | Backup |

安裝Keepalived：
1. 官網下載 https://keepalived.org/ (此練習下載2.2.8版)，並上傳到主機上
2. `cd ~`
3. `mkdir keepalived`
4. `tar -zxf keepalived-2.2.8.tar.gz -C keepalived/`
5. `cd keepalived/keepalived-2.2.8/`
6. `./configure --sysconf=/etc --prefix=/usr/local`
7. `make && make install`

### 配置文件
`cp /etc/keepalived/keepalived.conf.sample /etc/keepalived/keepalived.conf`
`vim /etc/keepalived/keepalived.conf`
```
! Configuration File for keepalived

#全局設定
global_defs {
   notification_email {
     #接收皆換通知的收件email
   }
   notification_email_from {發信者}
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   #運行keepalived服務器的一個標示，可以用作發送郵件的主題信息，這裡配置跟主機名稱相同即可
   router_id keepalived1
   #默認是不跳過檢查。檢查收到的VRRP通告中的所有地址可能會比較耗時，設置此命令的意思是，如果通告與接收的上一個通告來自相同的master路由器，則不執行檢查(跳過檢查)
   vrrp_skip_check_adv_addr
   #嚴格遵守VRRP協議
   vrrp_strict
   #在一個接手發送的兩個免費ARP之間的延遲，可以精準到毫秒級，默認為0
   vrrp_garp_interval 0
   #在一個網卡上每組va消息之間的延遲時間，默認為0
   vrrp_gna_interval 0
}

#VRRP相關設定
vrrp_instance VI_1 {
    #MASTER|BACKUP
    state MASTER
	#VRRP實例綁定的接口，用於發送VRRP對應的消息包，使用當前服務器的網卡名稱即可
    interface ens33
	#VRRP實例的id值，介於0~255，要聯網的文件共用一個id
    virtual_router_id 51
	#優先級
    priority 100
	#發送VRRP通告的時間間隔，默認1(單位秒)
    advert_int 1
    authentication {
		#驗證方式，默認密碼方式
        auth_type PASS
		#密碼，最多8位
        auth_pass 1111
    }
	#可以多組
    virtual_ipaddress {
        192.168.200.222 #要聯網的文件共用一個ip
    }
}

#LVS相關設定，這邊就不關注
virtual_server 192.168.200.100 443 {
...
```

啟動keepalived
1. `cd /usr/local/sbin/`
2.  `./keepalived`

keepalived script
```
vrrp_script 腳本名稱
{
    script "腳本位置"
    interval 3 #執行時間間隔
    weight -20 #動態調整vrrp_instance的優先級
}
```
1. `vim /etc/keepalived/ck_nginx.sh`
```
#!/bin/bash
num=`ps -C nginx --no-header | wc -l`
if [ $num -eq 0 ]; then
 /usr/local/nginx/sbin/nginx
 sleep 2
 if [ `ps -C nginx --no-header | wc -l` -wq 0 ]; then
  killall keepalived
 fi
fi
```
2. `chmod  755 /etc/keepalived/ck_nginx.sh`
3. `vim /etc/keepalived/keepalived.conf` 寫在vrrp_instance VI_1 之前與之同級
```
vrrp_script ck_nginx{
    script "/etc/keepalived/ck_nginx.sh"
    interval 3
    weight -20
}
vrrp_instance VI_1 {
    state MASTER #如果要實現這樣的HA，state 都改為BACKUP(先啟動的會被認定MASTER)
    #然後都是非搶占，意味著就算主機修復也不主動爭奪
    #nopreempt
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.200.222
    }
    #另外也要加上這個
    track_script{
        ck_nginx
    }    
}
```

## 下載站點
透過ngx_http_autoindex_module 實現，該模塊處理以斜槓("/")結尾的請求，並生成目錄列表；此模塊在編譯時就會自動加載
* `autoindex on|off` 開啟目錄列表輸出，默認off；寫在location 塊
* `autoindex_exact_size on|off` 對應HTML 格式，指定是否在列表展示文件的詳細大小，默認on(單位bytes)、off則是顯示大概大小(單位kb 或MB 或GB)
* `autoindex_format html|xml|json|jsonp` 目錄列表的格式，默認html
* `autoindex_localtime on|off` 對應HTML 格式，顯示時間，默認off(GMT 時間)、on則是文件的服務器時間

example:
```
location /download {
    root /usr/local;
    autoindex on;
    autoindex_exact_size on;
    autoindex_format html;
    autoindex_localtime on;
}
```

## 認證
透過ngx_http_auth_basic_module 實現，允許通過使用"HTTP 基本身分驗證"協議驗證用戶名及密碼來限制對資源的訪問。此模塊默認就會自動加載，如果不需要則使用參數--without-http_auth_basic_module
* `auth_basic string|off` 啟用驗證(將字符串返回客戶端做提示)，默認off；寫在location 塊
* `auth_basic_user_file file` 指定用戶名及密碼所在文件

example:
```
location /download {
    root /usr/local;
    autoindex on;
    autoindex_exact_size on;
    autoindex_format html;
    autoindex_localtime on;
    auth_basic 'please input your auth';
    auth_basic_user_file htpassword;
}
```
用戶名及密碼所在文件需要使用htpassword 工具生成
* `yum install -y httpd-tools`
* `htpasswd -c /usr/local/nginx/conf/htpassword username` 創建新文件記錄用戶名和密碼
* `htpasswd -b /usr/local/nginx/conf/htpassword username password` 文件中新增用戶名和密碼
* `htpasswd -D /usr/local/nginx/conf/htpassword username` 刪除文件中用戶
* `htpasswd -v /usr/local/nginx/conf/htpassword username` 驗證文件中用戶