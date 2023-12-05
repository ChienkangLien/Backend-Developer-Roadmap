# OpenID
OpenID 是一個去中心化的網上身分認證系統，也是基於 OAuth 2.0 規範框架的可互通身份驗證協定。對於支援 OpenID 的網站，使用者不需要記住像使用者名稱和密碼這樣的傳統驗證標記。
取而代之的是，他們只需要預先在一個作為 OpenID 身分提供者（identity provider, IdP）的網站上註冊。任何網站都可以使用 OpenID 來作為使用者登入的一種方式，任何網站也都可以作為 OpenID 身分提供者。
## 基本術語
* 終端使用者（End User）：想要向某個網站表明身分的人。
* 標識（Identifier）：終端使用者用以標識其身分的URL或XRI。
* 身分提供者（Identity Provider, IdP）：提供OpenID URL或XRI註冊和驗證服務的服務提供者。
* 依賴方（Relying Party, RP）：想要對終端使用者的標識進行驗證的網站。

## 登入流程
1. 有一登入網站，比如 example.com，在登入表單中連接了 OpenID 客戶端。
2. 使用者 Alice 在身分提供者 openid-provider.org 註冊了一個 OpenID 標識：`alice.openid-provider.org`。她到 example.com 去將`alice.openid-provider.org`填入 OpenID 登入表單。

如果標識是一個URL，依賴方 example.com 做的第一件事情是將這個URL轉換為典型格式：http://alice.openid-provider.org/ 。
* 在OpenID 1.0中，依賴方請求該URL對應的網頁，並發現提供者伺服器，比如 http://openid-provider.org/openid-auth.php 。
* 從OpenID 2.0開始，依賴方請求的是XRDS文件（也稱為Yadis文件），使用內容類型 application/xrds+xml。

3. 依賴方可以通過兩種模式來與身分提供者通訊：
* checkid_immediate：兩個伺服器間的所有通訊都在後台進行，不提示使用者。
* checkid_setup：使用者使用訪問依賴方站點的同一個瀏覽器窗口與身分提供者伺服器互動。

第二種模式更加常用。而且，如果操作不能夠自動進行的話， checkid_immediate 模式會轉換為 checkid_setup 模式。

4. 依賴方與身分提供者建立一個「共享密鑰」——一個聯絡控制代碼，然後依賴方儲存它。如果使用 checkid_setup 模式，依賴方將使用者的網頁瀏覽器重新導向到提供者。在這個例子裡， Alice 的瀏覽器被重新導向到 openid-provider.org ，來向提供者驗證自己。
5. 如果Alice當前沒有登入到 openid-provider.org ，她可能被提示輸入密碼，然後被問到是否信任依賴方。
6. 但是在這個階段，example.com 不能夠確定收到的資訊是否來自於 openid-provider.org。如果他們之前建立了「共享密鑰」，依賴方就可以用它來驗證收到的資訊。

這樣一個依賴方被稱為是stateful的，因為它儲存了會話間的「共享密鑰」。作為對比，stateless的驗證方必須再作一次背景請求（check_authentication）來確保資料的確來自 openid-provider.org。

7. Alice的標識被驗證之後，她被看作以 alice.openid-provider.org 登入到 example.com 。接著，這個站點可以儲存這次會話，或者，如果這是她的第一次登入，提示她輸入一些專門針對 example.com 的資訊，以完成註冊。

### 共享密鑰
上述登入流程中的共享密鑰、也就是 ID Token ，採取了 JWT 的規格。