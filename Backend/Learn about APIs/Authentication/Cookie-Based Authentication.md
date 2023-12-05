# Cookie-Based Authentication
Cookie 是瀏覽網站時由網路伺服器建立並由網頁瀏覽器存放在使用者電腦或其他裝置的小文字檔案。

Cookie 使Web 伺服器能在使用者的裝置儲存狀態資訊（如添加到線上商店購物車中的商品）或跟蹤使用者的瀏覽活動（如點擊特定按鈕、登入或記錄歷史）。

## 工作原理
1. 用戶登錄過程：
* 用戶登錄： 用戶提供身份信息（如用戶名和密碼），服務器驗證身份，並在驗證成功後創建一個會話（Session）。
* 設置 Cookie： 服務器生成一個包含會話標識信息（通常是一個唯一的 Session ID）的 Cookie，並在 HTTP 響應中將該 Cookie 發送到客戶端的瀏覽器中。

2. Cookie 存儲和傳輸：
* 存儲在客戶端： 當客戶端收到來自服務器的響應時，其中包含了設置的 Cookie。瀏覽器會將 Cookie 存儲在用戶的本地系統中。
* 自動發送： 每次用戶發送請求時，瀏覽器都會自動將相關域名下的 Cookie 信息附加到請求頭中，並發送給服務器。

3. 身份驗證過程：
* 請求處理： 服務器收到請求時，會檢查請求中的 Cookie，尋找包含會話標識信息的 Cookie。
* 驗證身份： 服務器使用 Cookie 中的會話標識信息來查找相應的會話數據。它驗證該標識是否有效且與有效的用戶會話相關聯。
* 身份確認： 如果服務器找到了有效的會話並確認會話有效，就可以確認請求的用戶身份，並根據權限控制等進行相應的操作。

## 與服務器端的搭配
Session 和 Cookie 不是同一種東西，它們是兩個相關但不同的概念，通常會結合使用。

* Cookie：
定義： Cookie 是在客戶端（瀏覽器）存儲數據的一種方式，由服務器通過 HTTP 響應頭發送給客戶端，並存儲在客戶端的本地文件中。
用途： Cookie 主要用於在客戶端存儲少量的數據，如會話標識、用戶偏好設置等。

* Session：
定義： Session 是服務器端的一個會話（或會話狀態），用於存儲特定用戶的信息和數據。
用途： Session 主要用於在服務器端存儲用戶的狀態信息，例如用戶的登錄狀態、購物車內容等。

關系：
通常情況下，服務器在創建會話時會生成一個會話標識（Session ID），並將該會話標識發送給客戶端的瀏覽器。這個會話標識通常以 Cookie 的形式存儲在客戶端。

當客戶端發送請求時，瀏覽器會自動附帶包含會話標識的 Cookie，服務器端利用這個會話標識來識別對應的會話信息。

注意：Java Servlet 中使用 request.getSession() 會自動處理會話和 Session ID 的管理，無需顯式處理 Session ID、也就不須操作Cookie 。

## 過期時間管理
* Cookie 的過期時間設置：
在設置 Cookie 時，可以通過設置 Expires(指定一個具體的過期日期/時間) 或 Max-Age(指定 Cookie 過期的秒數) 屬性來定義 Cookie 的過期時間。

* Session 的過期時間設置：
在服務器端設置 Session 的過期時間。具體設置方法取決於使用的編程語言和框架。 

## 安全性考慮
* 跨域請求：
瀏覽器將在跨域請求時限制對其他域的 Cookie 存取。通常情況下，Cookie 是與特定域名（或子域名）綁定的，因此來自其他域的請求無法訪問該 Cookie，這會導致無法成功通過身份驗證。即使使用 CORS（跨來源資源共享） 進行配置的情況下，，瀏覽器也可能不會在跨域請求中包含 Cookie。
* CSRF 攻擊防範：
SameSite 屬性設置：
同站點策略（SameSite）是一種 Cookie 屬性，用於防止某些類型的 CSRF 攻擊。它有三個可能的值：Strict、Lax 和 None。
在設置 Cookie 時，通過設置 SameSite 屬性為 Strict 或 Lax 可以減少 CSRF 攻擊的風險。

* 安全標志和策略：
Secure 和 HttpOnly 標志設置：
Secure 標志確保 Cookie 只在加密的 HTTPS 連接下傳輸。
HttpOnly 標志阻止 JavaScript 訪問 Cookie 的內容，這樣可以防止 XSS（跨站腳本攻擊）。

```java=
HttpServletResponse response = ...; // 獲取 HttpServletResponse 對象
Cookie cookie = new Cookie("sessionCookie", "sessionValue");
cookie.setSecure(true); // 只在 HTTPS 連接下發送 Cookie
cookie.setHttpOnly(true); // 阻止 JavaScript 訪問此 Cookie
cookie.setSameSite(Cookie.SameSite.STRICT); // 設置 SameSite 屬性為 Strict
response.addCookie(cookie); // 將 Cookie 添加到響應中
```

### CSRF 攻擊
CSRF（Cross-Site Request Forgery）跨站請求偽造是一種常見的網絡攻擊，攻擊者利用受害者在已登錄的情況下，誘使其在未經意識的情況下發送惡意請求。攻擊者通過在受害者身份驗證的網站上植入惡意代碼或鏈接，利用受害者的登錄狀態執行某些特定的操作（如轉賬、更改密碼等）。

例如，一個攻擊者可能在一個論壇中發布了一篇帶有惡意鏈接的帖子。如果受害者登錄了自己的銀行賬戶，並且點擊了該帖子中的鏈接，這個鏈接可能會觸發一個銀行轉賬請求或者其他敏感操作，因為瀏覽器攜帶了受害者在銀行網站的身份認證信息。

防範 CSRF 攻擊的常見方法包括使用 CSRF Token、檢查 Referer 頭部、使用 SameSite Cookie 屬性等。這些方法都有助於在請求中加入額外的驗證信息，確保請求來自合法的源，並且不是由惡意代碼或鏈接觸發的。

### 跨域請求：
瀏覽器的同源政策（Same-Origin Policy），限制了來自不同源的 JavaScript 代碼對網頁的存取。簡單來說，當網頁上的 JavaScript 代碼試圖訪問不同來源的資源時（例如，從 https://domain1.com 試圖訪問 https://domain2.com 的資源），瀏覽器會阻止這樣的跨域請求。