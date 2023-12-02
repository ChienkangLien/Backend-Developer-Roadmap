# REST
REpresentational State Transfer，是一種架構風格，並沒有明確的規範。而符合這個風格設計的API 也可以叫做RESTful API。
## 客戶端與伺服器分離
客戶端的實作和伺服器的實作可以獨立完成，無需互相了解(無狀態)。
只要每一方都知道要向對方發送什麼格式的訊息，它們就可以保持模組化和獨立。將使用者介面問題與資料儲存問題分開，提高了跨平台介面的靈活性。

## 資源導向
REST 將所有的系統功能抽象為資源（Resource），每個資源都有唯一的標識符（URI），並且用不同的 HTTP 方法來操作這些資源。

並使用統一的接口（Uniform Interface）來對資源進行操作，包括資源的標識、操作資源的方式、資源的表示形式(如 JSON、XML)等。

## 客戶端與伺服器之間的通信
通常基於 HTTP 協議
### 請求
REST 要求客戶端向伺服器發出請求，以便檢索或修改伺服器上的資料。請求通常包括以下內容：

* HTTP 動詞，定義要執行的動作類型
* header ，允許客戶端傳遞有關請求的訊息
* 資源的路徑
* 訊息正文(可選的)

#### HTTP 動詞
GET — 從指定的資源獲取資訊。對於無副作用的、只需要獲取資料的請求使用這個方法。
POST — 用於提交資料，通常用於創建新資源。可能導致新的資源的創建和修改。
PUT — 更新指定的資源，用新的資料替換指定資源的所有當前表示。
DELETE — 用於刪除指定的資源。

和以下其他方法

PATCH — 不同於PUT，PATCH 通常用於只更新資源的部分內容。
HEAD — 類似於GET，但是只返回資源的標頭信息，不返回實際資源的內容。
OPTIONS — 當客戶端不確定可以使用哪些方法時，可以向服務器發送 OPTIONS 請求以獲取支持的方法列表。
TRACE — 用於獲取客戶端與服務器之間的通信環回的資源的內容。不常用且潛在的安全風險，通常被禁用。
CONNECT — 用於告知服務器與客戶端之間的連接即將建立。通常用於代理伺服器。

#### 標頭中的 Accept
客戶端發送它能夠從伺服器接收的內容類型，確保伺服器不會發送客戶端無法理解或處理的資料。

通常以MIME 類型（Multipurpose Internet Mail Extensions），來指定所需的回應資源類型(type/a subtype)。

例如，包含 HTML 的文字檔案將使用類型指定text/html、如果此文字檔案包含 CSS，則它將被指定為text/css、通用文字檔將表示為text/plain。
如果客戶端正在等待text/css 但是收到text/plain，它將無法識別該內容。

其他類型：
image — image/png、image/jpeg、image/gif
audio — audio/wav、audio/mpeg
video — video/mp4、video/ogg
application — application/json、application/pdf、application/xml、application/octet-stream

#### 路徑
在 RESTful API 中，路徑的設計應可協助客戶端了解正在發生的情況。按照慣例，路徑的第一部分應該是資源的複數形式。

`fashionboutique.com/customers/223/orders/12 (GET)`
我們正在為223 的客戶存取id 為12 的訂單

當引用資源為列表或集合時，並不總是需要添加id。
`fashionboutique.com/customers (POST)`
不需要額外的標識符，因為伺服器將為新物件產生一個id

檢索具有指定的資源
`fashionboutique.com/customers/:id (GET)`
刪除指定資源中的項目
`fashionboutique.com/customers/:id (DELETE)`
### 回應
#### 標頭中的 Content-Type
伺服器說明發送的內容類型，通常也是MIME 類型
#### 回應代碼
提醒客戶端有關操作的資訊。常見的狀態代碼及其使用方式：

200 (OK) — HTTP 請求成功的標準回應。
201 (CREATED) — 成功建立專案的標準回應。
204 (NO CONTENT) — 請求成功，而在回應正文中不會傳回任何內容。
400 (BAD REQUEST) — 由於請求語法錯誤、大小過大或其他客戶端錯誤，無法處理請求。
403 (FORBIDDEN) — 客戶端無權存取該資源。
404 (NOT FOUND) — 目前無法找到該資源。
500 (INTERNAL SERVER ERROR) — 如果沒有更具體的可用信息，則為意外故障的一般答案。

對於每個 HTTP 動詞，伺服器在成功時應傳回預期的狀態代碼：
GET — 返回 200
POST — 返回 201
PUT — 返回 200
DELETE — 傳回 204
如果操作失敗，則傳回與遇到的問題相對應的最具體的狀態代碼。
