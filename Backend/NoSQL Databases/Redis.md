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

安裝目錄下
redis-benchmark：性能測試工具
redis-check-aof：修復有問題的aof 文件
redis-check-rdb：修復有問題的rdb 文件
redis-sentinel：集群使用
redis-server：服務器啟動命令
redis-cli：客戶端，操作入口

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
6. `kill -9 進程號` 關閉(強制但是不安全)，或是在命令行客户端中使用`shutdown`

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
使用RedisDesktopManager(非官方提供)
https://github.com/lework/RedisDesktopManager-Windows
這裡下載2022.5 ，解壓後即可安裝

### Java 客戶端
Jedis、SpringDataRedis

## 命令
以下舉例一些常用的
### key
* KEYS pattern：查找所有符合給定模式(pattern)的key
* DEL key：若key 存在時刪除key
* EXISTS key：檢查key 是否存在
* EXPIRE key seconds：為key 設置過期時間，以秒計
* TTL key：查看過期時間
* TYPE key：查看key 類型
* UNLINK key：非阻塞刪除，先將key 從keyspace 元數據中刪除、真正的刪除會在後續異步執行

#### 庫
* SELECT：切換庫(默認使用0)
* DBSIZE：查看當前庫的數量
* FLUSHDB：清空當前庫
* FLUSHALL：清空所有庫

### String
其value 是字符串，不過根據字符串格式，可分三類：string(普通字符串)、int(整數，可做自增減)、float(浮點數，可做自增減)；底層都是字節數組型式存儲，只不過是編碼方式不同(最大空間不能超過512m)

* SET：添加或者修改已經存在的一個String 類型的鍵值對
* GET：根據key 獲取String 類型的value
* APPEND：將給定的value 追加到原值末尾
* STRLEN：獲取值的長度
* MSET：批量添加多個String 類型的鍵值對
* MGET：根據多個key 獲取多個String 類型的value
* MSET key value [key value ...]：同時設置一個或多個 key-value 對，有原子性
* INCR(DECR)：讓一個整型的key 自增減1
* INCRBY(DECRBY):讓一個整型的key 自增減並指定步長，例如：incrby num 2 讓num值自增2
* INCRBYFLOAT(DECRBYFLOAT)：讓一個浮點類型的數字自增並指定步長
* SETNX：添加一個String 類型的鍵值對，前提是這個key 不存在，否則不執行
* SETEX：添加一個String 類型的鍵值對，並且指定有效期
* GETRANGE key <起始位置> <結束位置>：獲取key 的起始位置到結束位置的值
* SETRANGE key <起始位置> value：將value 的值覆蓋起始位置開始
* GETSET：用新值換舊值，並取得舊值

Redis 的key 允許有多個單詞形成層級結構，多個單詞之間用':'隔開，例如：項目名:業務名:類型:id。
例如我們的項目名稱叫foo，有user 和product 兩種不同類型的數據，我們可以
這樣定義key：
* user相關的key：foo:user:1
* product相關的key：foo:product:1

### Hash
也叫散列，其value 是一個無序字典，類似於Java 中的HashMap 結構。
String 結構是將對象序列化為JSON 字符串後存儲，當需要修改對象某個字段時很不方便
![image](https://hackmd.io/_uploads/HyQDdgXQA.png)
而Hash 結構可以將對象中的每個字段獨立存儲，可以針對單個字段做CRUD。
![image](https://hackmd.io/_uploads/B11OOlX7C.png)

* HSET key field value：添加或者修改hash 類型key 的field 的值
* HGET key field：獲取一個hash 類型key 的field 的值
* HEXISTS key field：查看key 中，指定的field 是否存在
* HMSET：批量添加多個hash 類型key 的field 的值
* HMGET：批量獲取多個hash 類型key 的field 的值
* HGETALL：獲取一個hash 類型的key 中的所有的field 和value
* HKEYS：獲取一個hash 類型的key 中的所有的field
* HVALS：獲取一個hash 類型的key 中的所有的value
* HINCRBY:讓一個hash 類型key 的字段值自增並指定步長
* HSETNX：添加一個hash 類型的key 的field 值，前提是這個field 不存在，否則不執行

### List
與Java 中的LinkedList 類似，可以看做是一個雙向鏈表結構。既可以支持正向檢索和也可以支持反向檢索。特征也與LinkedList 類似：有序、元素可以重復、插入和刪除快、查詢速度一般。

* LPUSH key element ...：向列表左側插入一個或多個元素
* LPOP key：移除並返回列表左側的第一個元素，沒有則返回nil，值光鍵亡
* RPUSH key element ...：向列表右側插入一個或多個元素
* RPOP key：移除並返回列表右側的第一個元素，值光鍵亡
* LRANGE key star end：返回一段角標範圍內的所有元素
* LINDEX key index：通過索引獲取列表中的元素
* BLPOP 和BRPOP：與LPOP 和RPOP 類似，只不過在沒有元素時等待指定時間，而不是直接返回nil
* RPOPLPUSH source destination：移除列表的最後一個元素，並將該元素添加到另一個列表並返回
* LINSERT key BEFORE|AFTER value newvalue：在列表的元素前或者後插入元素
* LLEN key：獲取列表長度
* LSET key index value：通過索引設置列表元素的值
* LREM key count value：移除列表元素

![image](https://hackmd.io/_uploads/H1VVqlQ7A.png)

### Set
與Java 中的HashSet 類似，可以看做是一個value 為null 的HashMap。因為也是一個hash 表，因此具備與HashSet 類似的特征：無序、元素不可重復、查找快、支持交集和聯集和差集等功能。

* SADD key member ...：向set 中添加一個或多個元素
* SREM key member ... :移除set 中的指定元素
* SCARD key：返回set 中元素的個數
* SISMEMBER key member：判斷一個元素是否存在於set 中
* SMEMBERS：獲取set 中的所有元素
* SPOP key：移除並返回集合中的一個隨機元素
* SRANDMEMBER key [count]：返回集合中一個或多個隨機數
* SMOVE source destination member：將member 元素從source 集合移動到destination 集合
* SINTER key1 key2 ...：求key1 與key2 的交集
* SDIFF key1 key2 ...：求key1 與key2 的差集(key1 中的，不存在key2 中)
* SUNION key1 key2 ...：求key1 和key2 的聯集

![image](https://hackmd.io/_uploads/BJiRsx7XR.png)

### SortedSet(Zset)
可排序的set 集合，與Java 中的TreeSet 有些類似，但底層數據結構卻差別很大。SortedSet 中的每一個元素都帶有一個score 屬性，可以基於score 屬性對元素排序，底層的實現是一個跳表（SkipList）加hash 表。具備特性：可排序、元素不重復、查詢速度快。

* ZADD key score member：添加一個或多個元素到sorted set，如果已經存在則更新其score 值
* ZREM key member：刪除sorted set 中的一個指定元素
* ZSCORE key member :獲取sorted set 中的指定元素的score 值
* ZRANK key member：獲取sorted set 中的指定元素的排名
* ZCARD key：獲取sorted set 中的元素個數
* ZCOUNT key min max：統計score 值在給定範圍內的所有元素的個數
* ZINCRBY key increment member：讓sorted set 中的指定元素自增，步長為指定的increment 值
* ZRANGE key min max [WITHSCORES]：按照score 排序後，獲取指定排名範圍內的元素
* ZRANGEBYSCORE key min max [WITHSCORES]：按照score排序後，獲取指定score 範圍內的元素
* ZDIFF、ZINTER、ZUNION：求差集、交集、聯集

注意：所有的排名默認都是升序，如果要降序則在命令的Z 後面添加REV 即可。

### Bitmaps
本身是一個字符串，不是數據類型，但是它可以對字符串的位進行操作。可以把Bitmaps 想像成一個以位為單位的數組，數組的每個單元只能存放0和1，數組的下標在Bitmaps 中稱為偏移量(從0開始)。合理使用操作位可以有效地提高內存使用率和開發使用率。

* SETBIT key offset value：設置某個偏移量的值(0或1)
* GETBIT key offset：獲取某個偏移量的值
* BITCOUNT key [start] [end]：統計值為1的bit 數
* BITOP AND/OR/NOT/XOR destkey [key...]：將多個Bitmaps 進行and(交集)、or(聯集)、not(非)、xor(抑或)操作，並把結果保存在destkey 中

### HyperLogLog
只會根據輸入元素來計算基數，而不會儲存輸入元素本身，不能像集合那樣，返回輸入的各個元素。假如資料集{1,3,5,7,5,7,8}、那麼這個資料集的基數集為{1,3,5,7,8}，基數(不重複元素)為5。

* PFADD key element...：向HLL 加元素
* PFCOUNT key...：計算基數
* PFMERGE destkey sourcekey...：將多個HLL 合併後的結果存到另一個HLL

### Geographic
提供經緯度設置，查詢範圍，距離查詢等。有效經度-180到180、緯度-85.05112878到85.05112878、超出範圍會返回錯誤。

* GEOADD key longitude latitude member：添加地理位置
* GEOPOS key member：獲得座標值
* GEODIST key member1 member2 [m|km|ft|mi]：取得兩個位置的直線距離(單位默認為m)
* GEORADIUS key longitude latitude radius m|km|ft|mi：以給定的經緯度為中心，找出某一半徑內的元素

## 發布和訂閱
Redis 客戶端可以訂閱任意數量的頻道
第一個客戶端，訂閱
```
redis 127.0.0.1:6379> SUBSCRIBE mych

Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "mych"
3) (integer) 1
```
第二個客戶端，發送
```
redis 127.0.0.1:6379> PUBLISH mych "Learn redis"

(integer) 1

# 訂閱者的客戶端會顯示如下消息
 1) "message"
2) "mych"
3) "Learn redis"
```
