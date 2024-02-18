# MongoDB
支持的數據結構非常鬆散，以一種類似於JSON 的格式叫BSON 的文件來儲存資料，並支援動態查詢。

## 結構
| SQL術語/概念 | MongoDB術語/概念 | 解釋/說明 |
| -------- | -------- | -------- |
| database | database | 資料庫 |
|table | collection | 資料庫表/集合 |
|row | document | 資料記錄行/文檔 |
|column	| field | 資料字段/域 |
|index | index | 索引 |
|table joins | | 表連接，MongoDB 不支持 |
| | 嵌入文檔 | MongoDB 通過嵌入式文檔來替代多表連接 |
|primary key | primary key | 主鍵，MongoDB自動將_id字段設置為主鍵 |

## 資料模型
最小存儲單位就是文檔(document)，對應於關係資料庫的行。資料在MongoDB 中以BSON（Binary-JSON）文檔的格式存儲。
BSON（Binary Serialized Document Format）是一種類json 的二進制形式存儲格式，簡稱Binary JSON。BSON和JSON一樣，支持內嵌的文檔物件和陣列物件，但是BSON 有JSON 沒有的一些數據類型，如Date 和Binary data 類型。


| 數據類型 | 描述 | 舉例 |
| -------- | -------- | -------- |	
| 字符串 | UTF-8字符串都可表示為字符串類型的數據 | {"x" : "foobar"} |
| 物件id | 物件id 是文檔的12字節的唯一 ID | {"x" : ObjectId()} |
| 布爾值 |	真或者假：true 或者false | {"x" : true} |
| 陣列 | 值的集合或者列表可以表示成陣列 | {"x" : ["a","b","c"]} |
| 32位整數 | 類型不可用。JavaScript 僅支持64位浮點數，所以32位整數會被自動轉換 | shell 是不支持該類型的，shell 中默認會轉換成64位浮點數 |
| 64位整數 | 不支持這個類型。shell 會使用一個特殊的內嵌文檔來顯示64位整數 | shell 是不支持該類型的，shell 中默認會轉換成64位浮點數 |
| 64位浮點數 | shell中的數字就是這一種類型	| {"x" : 3.14159, "y" : 3} |
| null | 表示空值或者未定義的物件 | {"x" : null} |	
| undefined | 文檔中也可以使用未定義類型	| {"x" : undefined} |
| 符號 | shell 不支持，shell 會將資料庫中的符號類型的資料自動轉換成字符串 | |
| 正則表達式 | 文檔中可以包含正則表達式，採用JavaScript 的正則表達式語法 | {"x" : /foobar/i} |	
| 代碼 | 文檔中還可以包含JavaScript代碼 | {"x" : function() { /* …… */ }} |
| 二進制數據 | 二進制數據可以由任意字節的串組成，不過shell 中無法使用 | |
| 最大值/最小值 | BSON 包括一個特殊類型，表示可能的最大值。shell 中沒有這個類型 | |

提示：
shell 默認使用64位浮點型數值。{"x" ： 3.14}或{"x" ： 3}。對於整型值，可以使用NumberInt（4字節符號整數）或NumberLong（8字節符號整數），{"x" : NumberInt("3")}
https://www.mongodb.com/docs/manual/reference/operator/query/type/#op._S_type

## 部屬
### windows
本地練習：
https://www.mongodb.com/try/download/community
下載.msi(7.0.5) 透過精靈安裝，即可操作UI 介面。

手動安裝：
下載zip (4.X.X)

方式一，命令行參數啟動：
1. 在解壓目錄中，手動建立一個目錄用於存放資料文件，如data/db
2. 在bin 目錄中打開命令行提示符，輸入如下命令`mongod --dbpath=..\data\db`

方式二，配置文件啟動：
1. 在解壓目錄中新建conf 文件夾，該文件夾中新建配置文件mongod.config ，內容參考如下(路徑與方式一相同)：
```yaml=
storage:
  #The directory where the mongod instance stores its data.Default Value is "\data\db" on Windows.
  dbPath: D:\mongodb-win32-x86_64-windows-7.0.5\data\db
```
2. 在bin 目錄中打開命令行提示符，輸入如下命令`mongod -f ../config/mongod.conf` 或 `mongod --config ../config/mongod.conf`

3. 在bin 目錄中打開命令行提示符，輸入如下命令即可連接
`mongo` 或 `mongo --host=127.0.0.1 --port=27017`

### linux
1. 下載.tgz 後丟到linux 中並解壓到當前目錄`tar -xvf mongodb-linux-x86_64-rhel70-4.0.28.tgz`
2. `mv mongodb-linux-x86_64-rhel70-4.0.28 /usr/local/mongodb`
3. 建立目錄
```shell=
#資料存儲目錄
mkdir -p /mongodb/single/data/db
#日誌存儲目錄
mkdir -p /mongodb/single/log
```
4. 新建配置文件`vi /mongodb/single/mongod.conf`
```shell=
systemLog:
    #MongoDB發送所有日誌輸出的目標指定為文件
    # #The path of the log file to which mongod or mongos should send all diagnostic logging information
    destination: file
    #mongod或mongos應向其發送所有診斷日誌記錄訊息的日誌文件的路徑
    path: "/mongodb/single/log/mongod.log"
    #當mongos或mongod實例重新啟動時，mongos或mongod會將新條目附加到現有日誌文件的末尾。
    logAppend: true
storage:
    #mongod實例存儲其資料的目錄。storage.dbPath設置僅適用於mongod。
    ##The directory where the mongod instance stores its data.Default Value is "/data/db".
    dbPath: "/mongodb/single/data/db"
    journal:
        #啟用或禁用持久性日誌以確保資料文件保持有效和可恢復。
        enabled: true
processManagement:
    #啟用在後台運行mongos或mongod進程的守護進程模式。
    fork: true
net:
    #服務實例綁定的IP，默認是localhost
    bindIp: localhost,192.168.191.133
    #bindIp，後接虛擬機的IP
    #綁定的端口，默認是27017
    port: 27017
```
5. 啟動`/usr/local/mongodb/bin/mongod -f /mongodb/single/mongod.conf`

## 基本指令
### 資料庫
* 選擇和創建資料庫：`use 資料庫名稱`，如果資料庫不存在則自動創建，ex: `use articledb`
* 查看有權限查看的所有的資料庫：`show dbs` 或 `show databases`
* 查看目前正在使用的資料庫：`db`
* 刪除庫：`db.dropDatabase()`

有一些庫名是保留的，可以直接訪問這些有特殊作用的資料庫。
1. admin： 從權限的角度來看，這是"root"庫。要是將一個用戶添加到這個庫，這個用戶自動繼承所有庫的權限。一些特定的服務器端命令也只能從這個庫運行，比如列出所有的庫或者關閉服務器。
2. local: 這個數據永遠不會被複製，可以用來存儲限於本地單台服務器的任意集合。
3. config: 當Mongo 用於分片設置時，config 庫在內部使用，用於保存分片的相關訊息。

### 集合
* 顯示創建：`db.createCollection(name)`，ex: `db.createCollection("mycollection")`
* 隱式創建：當向一個集合中插入一個文檔的時候，如果集合不存在，則會自動創建集合。通常使用隱式創建文檔即可。
* 查看：`show collections` 或 `show tables`
* 刪除：`db.集合.drop()`

### 文檔
* 新增：`db.<collection_name>.insertOne()` 添加一個文檔；`db.<collection_name>.insertMany()` 添加多個文檔
```shell=
db.collection.insert(
    <document or array of documents>,
    {
        writeConcern: <document>,
        ordered: <boolean>
    }
)
```
ex: 
```shell=
db.comment.insert({"articleid":"100000","content":"今天天氣真好，陽光明媚","userid":"1001","nickname":"Rose","createdatetime":new Date(),"likenum":NumberInt(10),"state":null})
```
以及
```shell=
try {
    db.comment.insertMany([
        {"_id":"1","articleid":"100001","content":"我們不應該把清晨浪費在手機上，健康很重要，一杯溫水幸福你我他。","userid":"1002","nickname":"相忘於江湖","createdatetime":new Date("2019-08-05T22:08:15.522Z"),"likenum":NumberInt(1000),"state":"1"},
        {"_id":"2","articleid":"100001","content":"我夏天空腹喝涼開水，冬天喝溫開水","userid":"1005","nickname":"伊人憔悴","createdatetime":new Date("2019-08-05T23:58:51.485Z"),"likenum":NumberInt(888),"state":"1"},
        {"_id":"3","articleid":"100001","content":"我一直喝涼開水，冬天夏天都喝。","userid":"1004","nickname":"傑克船長","createdatetime":new Date("2019-08-06T01:05:06.321Z"),"likenum":NumberInt(666),"state":"1"},
        {"_id":"4","articleid":"100001","content":"專家說不能空腹吃飯，影響健康。","userid":"1003","nickname":"凱撒","createdatetime":new Date("2019-08-06T08:18:35.288Z"),"likenum":NumberInt(2000),"state":"1"},
        {"_id":"5","articleid":"100001","content":"研究表明，剛燒開的水千萬不能喝，因為燙嘴。","userid":"1003","nickname":"凱撒","createdatetime":new Date("2019-08-06T11:01:02.521Z"),"likenum":NumberInt(3000),"state":"1"}
    ]);
} catch (e) {
	print (e);
}
```
以上批次新增也展示支援try catch。
參數
| Parameter | Type | Description |
| -------- | -------- | -------- |
| document | document or array | 要插入到集合中的文檔或文檔陣列（json格式） |
| writeConcern | document | 可選的。用於確定在進行寫操作時的確認級別，即確定資料已寫入到多少個副本中 |
| ordered | boolean | 可選。如果為true，則按順序插入陣列中的文檔，如果其中一個文檔出現錯誤，將返回而不處理陣列中的其餘文檔。如果為false，則執行無序插入，如果其中一個文檔出現錯誤，則繼續處理其他文檔。在版本2.6+中默認為true |

* 查詢：`db.<collection_name>.findOne()` 查第一個，`db.<collection_name>.find()` 查所有
```shell=
db.collection.find(<query>, [projection])
```
參數
| Parameter | Type | Description |
| -------- | -------- | -------- |
| query | document | 可選。使用查詢運算符指定選擇篩選器。若要返回集合中的所有文檔，則省略此參數或傳遞空文檔( {} )；ex: `db.comment.find({userid:"1003"})`(也可findOne) |
| projection | document | 可選。指定要在與查詢篩選器匹配的文檔中返回的字段（投影）。若要返回匹配文檔中的所有字段，則省略此參數；ex: `db.comment.find({userid:"1003"},{userid:1,nickname:1,_id:0})`|

* 修改：`db.<collection_name>.update(查詢對象, 新對象)` 默認情況下會使用新對象替換舊對象、ex: `db.comment.update({_id:"1"},{likenum:NumberInt(1001)})`，不建議使用。
加上$set 實現局部修改`db.<collection_name>.update(查詢對象, $set新對象)` ex: `db.comment.update({_id:"2"},{$set:{likenum:NumberInt(889)}})`。
也支持批量修改、ex: `db.comment.update({userid:"1003"},{$set:{nickname:"凯撒大帝"}},{multi:true})`。
原值增長修改，使用\$inc 運算符來實現(對3號數據的點讚數，每次遞增1)，ex: `db.comment.update({_id:"3"},{$inc:{likenum:NumberInt(1)}})`

詳細用法：
```shell=
db.collection.update(query, update, options)
//或
db.collection.update(
    <query>,
    <update>,
    {
        upsert: <boolean>,
        multi: <boolean>,
        writeConcern: <document>,
        collation: <document>,
        arrayFilters: [ <filterdocument1>, ... ],
        hint: <document|string> //Available starting in MongoDB 4.2
    }
)
```
參數
| Parameter | Type | Description |
| -------- | -------- | -------- |
| query | document | 更新的選擇條件。類似sql update查詢內where後面的內容 |
| update | document or pipeline | 要應用的修改。可以理解為sql update查詢內set後面的值 |
| upsert | boolean | 可選。如果設置為true，沒有與查詢條件匹配的文檔時將創建新文檔。默認值為false |
| multi | boolean | 可選。如果設置為true，則更新符合查詢條件的多個文檔。默認值為false，更新一個文檔 |
| writeConcern | document | 可選。確定寫入操作的確認級別。它指定了在MongoDB 寫入操作後，MongoDB 將向客戶端發送的確認數量，以及是否要等待確認的回覆 |
| collation | document | 指定比較和排序字符串時使用的規則。通常在需要進行特定語言或區域設置的情況下使用。它包含了一些設置，例如locale（語言環境）、caseLevel（區分大小寫）、caseFirst（字母排序位置）、numericOrdering（數字排序）、strength（比較強度）等 |
| arrayFilters | array | 可選。指定應該更新嵌套陣列中的哪些元素 |
| hint | document or string | 可選。指定MongoDB 在查找要更新的文檔時應該使用的索引。通常情況下，MongoDB 會自動選擇最佳的索引來執行查詢 |

* 刪除：`db.集合名稱.remove(條件)`，不帶條件則全刪。
* 統計：`db.collection.count(query, options)`、ex: `db.comment.count({userid:"1003"})`(也可不帶條件統計全部)

參數
| Parameter | Type | Description |
| -------- | -------- | -------- |
| query | document | 選擇條件 |
| options | document | 可選。用於修改計數的額外選項 |
* 分頁：`db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)`。
在find 方法後調用limit 來返回結果(TopN)，默認值20、ex: `db.comment.find().limit(3)`。
skip方法同樣接受一個數字參數作為跳過的記錄條數。（前N個不要），默認值是0、ex: `db.comment.find().skip(3)`。
```shell=
//第一頁
db.comment.find().skip(0).limit(2)
//第二頁
db.comment.find().skip(2).limit(2)
//第三頁
db.comment.find().skip(4).limit(2)
```
* 排序：sort() 方法可以通過參數指定排序的字段，並使用 1 和 -1 來指定排序的方式，其中 1 為升序排列，而 -1 是用於降序排列，ex: `db.comment.find().sort({userid:-1,likenum:1})`。
* 正則查詢：正則表達式是js 的語法(規則夾在//中)，ex: `db.comment.find({content:/^專家/})`。
* 比較查詢：
```shell=
db.集合名稱.find({ "field" : { $gt: value }}) // 大於: field > value
db.集合名稱.find({ "field" : { $lt: value }}) // 小於: field < value
db.集合名稱.find({ "field" : { $gte: value }}) // 大於等於: field >= value
db.集合名稱.find({ "field" : { $lte: value }}) // 小於等於: field <= value
db.集合名稱.find({ "field" : { $ne: value }}) // 不等於: field != value

db.comment.find({likenum:{$gt:NumberInt(700)}})
```
* 包含查詢：`db.comment.find({userid:{$in:["1003","1004"]}})`，如果是不包含則使用$nin。
* 條件查詢：
```shell=
$and:[ { },{ },{ } ]
db.comment.find({$and:[{likenum:{$gte:NumberInt(700)}},{likenum:{$lt:NumberInt(2000)}}]})

$or:[ { },{ },{ } ]
db.comment.find({$or:[ {userid:"1003"} ,{likenum:{$lt:1000} }]})
```

## 索引
1. 單字段索引，會自帶一個索引 _id
2. 覆合索引，多個字段組成。
3. 地理空間索引（Geospatial Index）：為了支持對地理空間坐標數據的有效查詢，MongoDB 提供了兩種特殊的索引：返回結果時使用平面幾何的二維索引和返回結果時使用球面幾何的二維球面索引。
4. 文本索引（Text Indexes）：MongoDB 提供了一種文本索引類型，支持在集合中搜索字符串內容。這些文本索引不存儲特定於語言的停止詞（例如"the"、"a"、"or"），而將集合中的詞作為詞幹，只存儲根詞。
5. 哈希索引（Hashed Indexes）：為了支持基於散列的分片，MongoDB 提供了散列索引類型，它對字段值的散列進行索引。這些索引在其範圍內的值分布更加隨機，但只支持相等匹配，不支持基於範圍的查詢。

* 查看：`db.collection.getIndexes()`
* 創建：`db.collection.createIndex(keys, options)`

參數
| Parameter | Type | Description |
| -------- | -------- | -------- |
| keys | document | 包含字段和值對的文檔，其中字段是索引鍵，值描述該字段的索引類型。對於字段上的升序索引，請指定值1；對於降序索引，請指定值-1。比如： {字段:1或-1} 。另外，MongoDB支持幾種不同的索引類型，包括文本、地理空間和哈希索引。 |
| options | document | 可選。包含一組控制索引創建的選項的文檔；常見的有unique(是否唯一、默認值為false)、name(索引的名稱)|
* 移除：`db.collection.dropIndex(index)`

### 執行計畫
分析查詢性能（Analyze Query Performance）通常使用執行計劃（解釋計劃、Explain Plan）來查看查詢的情況，如查詢耗費的時間、是否基於索引查詢等。`db.collection.find(query,options).explain(options)`
```shell=
> db.comment.find({userid:"1003"}).explain()
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "articledb.comment",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "userid" : {
                                "$eq" : "1003"
                        }
                },
                "winningPlan" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "userid" : {
                                        "$eq" : "1003"
                                }
                        },
                        "direction" : "forward"
                },
                "rejectedPlans" : [ ]
        },
        "serverInfo" : {
                "host" : "localhost.localdomain",
                "port" : 27017,
                "version" : "4.0.28",
                "gitVersion" : "af1a9dc12adcfa83cc19571cb3faba26eeddac92"
        },
        "ok" : 1
}
```
"stage" : "COLLSCAN"，表示全集合掃描。
而加入userid作為索引：
```shell=
> db.comment.find({userid:"1003"}).explain()
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "articledb.comment",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "userid" : {
                                "$eq" : "1003"
                        }
                },
                "winningPlan" : {
                        "stage" : "FETCH",
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "userid" : 1
                                },
                                "indexName" : "userid_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "userid" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "userid" : [
                                                "[\"1003\", \"1003\"]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "serverInfo" : {
                "host" : "localhost.localdomain",
                "port" : 27017,
                "version" : "4.0.28",
                "gitVersion" : "af1a9dc12adcfa83cc19571cb3faba26eeddac92"
        },
        "ok" : 1
}
```
"stage" : "IXSCAN"，基於索引的掃描。

而當查詢條件和查詢的投影僅包含索引字段時，MongoDB 會直接從索引返回結果，而不掃描任何文檔或將文檔帶入內存(省略FETCH 的步驟)。

## HA
MongoDB 的replicate 有兩種類型及三種腳色

兩種類型：
1. 主節點（Primary）類型：資料操作的主要連接點，可讀寫。
2. 次要（輔助、從）節點（Secondaries）類型：資料冗余備份節點，可以讀或選舉。

三種角色：
1. 主要成員（Primary）：主要接收所有寫操作，也就是主節點。
2. 副本成員（Replicate）：從主節點通過複製操作以維護相同的資料集，即備份資料，不可寫操作，但可以讀操作（但需要配置）。是默認的一種從節點類型。
3. 仲裁者（Arbiter）：不保留任何資料的副本，只具有投票選舉作用。當然也可以將仲裁服務器維護為副本集的一部分，即副本成員同時也可以是仲裁者。也是一種從節點類型。

### 架構實作
接下來操作一主一從一仲裁的架構(以port 號區分)，叢集名稱取作myrs
1. 建立資料夾
```shell=
mkdir -p /mongodb/replica_sets/myrs_27017/log \ &
mkdir -p /mongodb/replica_sets/myrs_27017/data/db
mkdir -p /mongodb/replica_sets/myrs_27018/log \ &
mkdir -p /mongodb/replica_sets/myrs_27018/data/db
mkdir -p /mongodb/replica_sets/myrs_27019/log \ &
mkdir -p /mongodb/replica_sets/myrs_27019/data/db
```
2. 配置文件
```shell=
vim /mongodb/replica_sets/myrs_27017/mongod.conf
```
將以下貼上
```shell=
systemLog:
    #MongoDB發送所有日誌輸出的目標指定為文件
    # #The path of the log file to which mongod or mongos should send all diagnostic logging information
    destination: file
    #mongod或mongos應向其發送所有診斷日誌記錄訊息的日誌文件的路徑
    path: "/mongodb/replica_sets/myrs_27017/log/mongod.log"
    #當mongos或mongod實例重新啟動時，mongos或mongod會將新條目附加到現有日誌文件的末尾。
    logAppend: true
storage:
    #mongod實例存儲其資料的目錄。storage.dbPath設置僅適用於mongod。
    ##The directory where the mongod instance stores its data.Default Value is "/data/db".
    dbPath: "/mongodb/replica_sets/myrs_27017/data/db"
    journal:
        #啟用或禁用持久性日誌以確保資料文件保持有效和可恢復。
        enabled: true
processManagement:
    #啟用在後台運行mongos或mongod進程的守護進程模式。
    fork: true
net:
    #服務實例綁定的IP，默認是localhost
    bindIp: localhost,192.168.191.133
    #bindIp，後接虛擬機的IP
    #綁定的端口，默認是27017
    port: 27017
replication:
    #副本集的名稱
    replSetName: myrs
```
```shell=
vim /mongodb/replica_sets/myrs_27018/mongod.conf
```
將以下貼上
```shell=
systemLog:
    #MongoDB發送所有日誌輸出的目標指定為文件
    # #The path of the log file to which mongod or mongos should send all diagnostic logging information
    destination: file
    #mongod或mongos應向其發送所有診斷日誌記錄訊息的日誌文件的路徑
    path: "/mongodb/replica_sets/myrs_27018/log/mongod.log"
    #當mongos或mongod實例重新啟動時，mongos或mongod會將新條目附加到現有日誌文件的末尾。
    logAppend: true
storage:
    #mongod實例存儲其資料的目錄。storage.dbPath設置僅適用於mongod。
    ##The directory where the mongod instance stores its data.Default Value is "/data/db".
    dbPath: "/mongodb/replica_sets/myrs_27018/data/db"
    journal:
        #啟用或禁用持久性日誌以確保資料文件保持有效和可恢復。
        enabled: true
processManagement:
    #啟用在後台運行mongos或mongod進程的守護進程模式。
    fork: true
net:
    #服務實例綁定的IP，默認是localhost
    bindIp: localhost,192.168.191.133
    #bindIp，後接虛擬機的IP
    #綁定的端口，默認是27017
    port: 27018
replication:
    #副本集的名稱
    replSetName: myrs
```
```shell=
vim /mongodb/replica_sets/myrs_27019/mongod.conf
```
將以下貼上
```shell=
systemLog:
    #MongoDB發送所有日誌輸出的目標指定為文件
    # #The path of the log file to which mongod or mongos should send all diagnostic logging information
    destination: file
    #mongod或mongos應向其發送所有診斷日誌記錄訊息的日誌文件的路徑
    path: "/mongodb/replica_sets/myrs_27019/log/mongod.log"
    #當mongos或mongod實例重新啟動時，mongos或mongod會將新條目附加到現有日誌文件的末尾。
    logAppend: true
storage:
    #mongod實例存儲其資料的目錄。storage.dbPath設置僅適用於mongod。
    ##The directory where the mongod instance stores its data.Default Value is "/data/db".
    dbPath: "/mongodb/replica_sets/myrs_27019/data/db"
    journal:
        #啟用或禁用持久性日誌以確保資料文件保持有效和可恢復。
        enabled: true
processManagement:
    #啟用在後台運行mongos或mongod進程的守護進程模式。
    fork: true
net:
    #服務實例綁定的IP，默認是localhost
    bindIp: localhost,192.168.191.133
    #bindIp，後接虛擬機的IP
    #綁定的端口，默認是27017
    port: 27019
replication:
    #副本集的名稱
    replSetName: myrs
```
3. 啟動
```shell=
/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/myrs_27017/mongod.conf

/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/myrs_27018/mongod.conf

/usr/local/mongodb/bin/mongod -f /mongodb/replica_sets/myrs_27019/mongod.conf
```

4. 初始化配置副本集和主節點：使用客戶端命令連接連接主節點(27017節點) `/usr/local/mongodb/bin/mongo` 進去之後
`rs.initiate(configuration)` 這裡就使用默認的配置就好，不帶參數

| Parameter | Type | Description |
| -------- | -------- | -------- |
| configuration | document | 可選的。指定新副本集配置文件。如果未指定配置，MongoDB 將使用預設的副本集配置 |
```shell=
> rs.initiate()
{
        "info2" : "no configuration specified. Using a default configuration for the set",
        "me" : "192.168.191.133:27017",
        "ok" : 1 // ok的值為1，說明創建成功
}
```
5. 加入從節點`rs.add(host, arbiterOnly)`

| Parameter | Type | Description |
| -------- | -------- | -------- |
| host | string or document | 要添加到副本集的新成員。 指定為字符串或配置文檔：1）如果是一個字符串，則指定新成員的主機名和可選的端口號；2）如果是一個文檔，指定在members 陣列中找到的副本集成員配置文檔；您必須在成員配置文檔中指定主機字段 |
| arbiterOnly | boolean | 可選的。 僅在host 值為字符串時適用。 如果為true，則添加的主機是仲裁者 |

以下是成員配置文檔範例：
```
{
    _id: <int>,
    host: <string>, // required
    arbiterOnly: <boolean>,
    buildIndexes: <boolean>,
    hidden: <boolean>,
    priority: <number>,
    tags: <document>,
    slaveDelay: <int>,
    votes: <number>
}
```
```shell=
> rs.add("192.168.191.133:27018")
{
        "ok" : 1,
        "operationTime" : Timestamp(1708263403, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1708263403, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
```
6. 加入仲裁節點，這裡使用另一個指令`rs.addArb(host)`
```shell=
> rs.addArb("192.168.191.133:27019")
{
        "ok" : 1,
        "operationTime" : Timestamp(1708263547, 2),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1708263547, 2),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
```

也可查看配置內容`rs.conf(configuration)`
```shell=
> rs.conf()
{
        "_id" : "myrs", // 副本集的配置資料存儲的主鍵值，默認就是副本集的名字
        "version" : 3,
        "protocolVersion" : NumberLong(1),
        "writeConcernMajorityJournalDefault" : true,
        "members" : [ // 副本集成員陣列
                {
                        "_id" : 0,
                        "host" : "192.168.191.133:27017",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 1,
                        "host" : "192.168.191.133:27018",
                        "arbiterOnly" : false,
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 1,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                },
                {
                        "_id" : 2,
                        "host" : "192.168.191.133:27019",
                        "arbiterOnly" : true, // 仲裁
                        "buildIndexes" : true,
                        "hidden" : false,
                        "priority" : 0,
                        "tags" : {

                        },
                        "slaveDelay" : NumberLong(0),
                        "votes" : 1
                }
        ],
        "settings" : {
                "chainingAllowed" : true,
                "heartbeatIntervalMillis" : 2000,
                "heartbeatTimeoutSecs" : 10,
                "electionTimeoutMillis" : 10000,
                "catchUpTimeoutMillis" : -1,
                "catchUpTakeoverDelayMillis" : 30000,
                "getLastErrorModes" : {

                },
                "getLastErrorDefaults" : {
                        "w" : 1,
                        "wtimeout" : 0
                },
                "replicaSetId" : ObjectId("65d20407f308e35b2881de12")
        }
}
```
或是`rs.status()` 查看叢集狀態。
```shell=
> rs.status()
{
        "set" : "myrs", // 叢集名稱
        "date" : ISODate("2024-02-18T13:41:36.819Z"),
        "myState" : 1, // 1 即狀態正常
        "term" : NumberLong(1),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1708263687, 1),
                        "t" : NumberLong(1)
                },
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1708263687, 1),
                        "t" : NumberLong(1)
                },
                "appliedOpTime" : {
                        "ts" : Timestamp(1708263687, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1708263687, 1),
                        "t" : NumberLong(1)
                }
        },
        "lastStableCheckpointTimestamp" : Timestamp(1708263667, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2024-02-18T13:20:07.160Z"),
                "electionTerm" : NumberLong(1),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(0, 0),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1708262407, 1),
                        "t" : NumberLong(-1)
                },
                "numVotesNeeded" : 1,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "newTermStartDate" : ISODate("2024-02-18T13:20:07.162Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2024-02-18T13:20:07.191Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "192.168.191.133:27017",
                        "health" : 1, // 1 即健康
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 1734,
                        "optime" : {
                                "ts" : Timestamp(1708263687, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2024-02-18T13:41:27Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1708262407, 2),
                        "electionDate" : ISODate("2024-02-18T13:20:07Z"),
                        "configVersion" : 3,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                },
                {
                        "_id" : 1,
                        "name" : "192.168.191.133:27018",
                        "health" : 1,
                        "state" : 2,
                        "stateStr" : "SECONDARY",
                        "uptime" : 293,
                        "optime" : {
                                "ts" : Timestamp(1708263687, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDurable" : {
                                "ts" : Timestamp(1708263687, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2024-02-18T13:41:27Z"),
                        "optimeDurableDate" : ISODate("2024-02-18T13:41:27Z"),
                        "lastHeartbeat" : ISODate("2024-02-18T13:41:35.405Z"),
                        "lastHeartbeatRecv" : ISODate("2024-02-18T13:41:36.412Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "192.168.191.133:27017",
                        "syncSourceHost" : "192.168.191.133:27017",
                        "syncSourceId" : 0,
                        "infoMessage" : "",
                        "configVersion" : 3
                },
                {
                        "_id" : 2,
                        "name" : "192.168.191.133:27019",
                        "health" : 1,
                        "state" : 7,
                        "stateStr" : "ARBITER",
                        "uptime" : 149,
                        "lastHeartbeat" : ISODate("2024-02-18T13:41:35.405Z"),
                        "lastHeartbeatRecv" : ISODate("2024-02-18T13:41:35.408Z"),
                        "pingMs" : NumberLong(0),
                        "lastHeartbeatMessage" : "",
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "configVersion" : 3
                }
        ],
        "ok" : 1,
        "operationTime" : Timestamp(1708263687, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1708263687, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
```

#### 副本集的操作
試著在主節點(27017)新增資料連接客戶端
```shell=
// 本機和27017、可省略host / port需告
[root@bobohost ~]# /usr/local/mongodb/bin/mongo
myrs:PRIMARY> use articledb
switched to db articledb
myrs:PRIMARY> db
articledb
myrs:PRIMARY> db.comment.insert({"articleid":"100000","content":"今天天氣真好，陽光明媚","userid":"1001","nickname":"Rose","createdatetime":new Date()})
WriteResult({ "nInserted" : 1 })
myrs:PRIMARY> db.comment.find()
{ "_id" : ObjectId("5d4d2ae3068138b4570f53bf"), "articleid" : "100000","content" : "今天天氣真好，陽光明媚", "userid" : "1001", "nickname" : "Rose","createdatetime" : ISODate("2019-08-09T08:12:19.427Z") }
```
接著使用從節點(27018)連接查看
```shell=
[root@bobohost ~]# /usr/local/mongodb/bin/mongo --port=27018
myrs:SECONDARY> show dbs
2024-02-18T06:05:03.593-0800 E QUERY    [js] Error: listDatabases failed:{
        "operationTime" : Timestamp(1708265097, 1),
        "ok" : 0,
        "errmsg" : "not master and slaveOk=false",
        "code" : 13435,
        "codeName" : "NotMasterNoSlaveOk",
        "$clusterTime" : {
                "clusterTime" : Timestamp(1708265097, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
Mongo.prototype.getDBs@src/mongo/shell/mongo.js:151:1
shellHelper.show@src/mongo/shell/utils.js:882:13
shellHelper@src/mongo/shell/utils.js:766:15
@(shellhelp2):1:1
```
有以上錯誤訊息是因為目前從節點只是一個備份，不是奴隸節點，無法讀取數據，寫當然更不行。
因為默認情況下，從節點是沒有讀寫權限的，可以增加讀的權限，但需要進行設置。`rs.slaveOk()` 或 `rs.slaveOk(true)`；新版本也升級為`rs.secondaryOk()`。

另外仲裁節點是沒有讀寫權限的，即便使用以上指令也無效。

另外注意的是：如果使用Compass 連接，不管是叢集的哪個節點，都可以做讀寫操作，因為背景Compass 會找出主節點並對其作業。

### 選舉原則
主節點選舉的觸發條件：
1. 主節點故障
2. 主節點網絡不可達（默認心跳訊息為10秒）
3. 人工幹預（rs.stepDown(600)）
一旦觸發選舉，就要根據一定規則來選主節點。

選舉規則是根據票數來決定誰獲勝：
* 票數最高，且獲得了"大多數"成員的投票支持的節點獲勝。假設集內投票成員數量為N，則大多數為 N/2 + 1。例如：3個投票成員，則大多數的值是2。當集內存活成員數量不足大多數時，將無法選舉出Primary，複製集將無法提供寫服務，處於只讀狀態。
* 若票數相同，且都獲得了"大多數"成員的投票支持的，資料新的節點獲勝。資料的新舊是通過操作日誌oplog 來對比的。

在獲得票數的時候，優先級（priority）參數影響重大。
可以通過設置優先級（priority）來設置額外票數。優先級即權重，取值為0-1000，相當於可額外增加0-1000的票數，優先級的值越大，就越可能獲得多數成員的投票（votes）數。指定較高的值可使成員更有資格成為主要成員，更低的值可使成員更不符合條件。而默認情況下，優先級的值是1。

以下是變更優先集的範例(連接近mongoDB 後操作)
1. 先將配置導入cfg變量`myrs:SECONDARY> cfg=rs.conf()`
2. 然後修改值（ID號默認從0開始）`myrs:SECONDARY> cfg.members[1].priority=2`
3. 重新加載配置`myrs:SECONDARY> rs.reconfig(cfg)`

為了避免選舉過程中出現平局的情況，建議複製集中的節點數量為奇數。如果只有兩個節點，一個是主節點，另一個是從節點，那麼添加一個仲裁節點可以使節點數量變為奇數，從而避免平局的發生。

服務降級：如果主節點以外的大多數節點故障，會讓複製集無法提供寫服務，處於只讀狀態。