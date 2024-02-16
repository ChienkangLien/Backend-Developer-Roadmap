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