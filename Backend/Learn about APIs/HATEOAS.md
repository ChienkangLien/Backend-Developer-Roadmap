# HATEOAS
REST成熟度大致上可以分成
1. Web服務只是使用 HTTP 作為傳輸方式，實際上只是遠程方法調用（RPC）的一種具體形式。SOAP 和 XML-RPC 都屬於此類。
2. Web服務引入了資源的概念，每個資源有對應的標識符和表達。
3. Web服務使用不同的 HTTP 方法來進行不同的操作，並且使用 HTTP 狀態碼來表示不同的結果。
4. Web服務使用 HATEOAS。在資源的表達中包含了連接資訊。客戶端可以根據連接來發現可以執行的動作。

超媒體即應用狀態引擎（Hypermedia as the Engine of Application State）是REST 的一個約束條件，伺服器通過超媒體（Hypermedia）動態提供信息。除了對超媒體的一般理解之外，REST客戶端與服務端仍是無狀態的。

## 概念
* Hypermedia（超媒體）：
在 HATEOAS 中，超媒體是指在 API 的回應中，除了數據之外，還包含了與之相關的超連結、動作和其他相關資訊。這些超媒體連結可以指引客戶端進行下一步操作或導向其他資源。
* Engine of Application State（應用狀態引擎）
HATEOAS 鼓勵使用超媒體來驅動應用程式的狀態轉換，即客戶端應該根據伺服器返回的超媒體來決定接下來的操作，而不是預先知道所有可用的操作。

```json=
HTTP/1.1 200 OK

{
    "account": {
        "account_number": 12345,
        "balance": {
            "currency": "usd",
            "value": 100.00
        },
        "links": {
            "deposits": "/accounts/12345/deposits",
            "withdrawals": "/accounts/12345/withdrawals",
            "transfers": "/accounts/12345/transfers",
            "close-requests": "/accounts/12345/close-requests"
        }
    }
}
```
響應包含後續可能訪問的連結：發布存款、取款、轉帳或關閉請求（關閉帳戶）。操作這些連結一樣是根據RESTful再發送對應的GET、PUT...請求。