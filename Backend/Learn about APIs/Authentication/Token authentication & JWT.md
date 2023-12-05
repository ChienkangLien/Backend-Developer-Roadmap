# Token authentication & JWT
token(令牌/權杖) 是由受信任的來源頒發的象徵性物品，類似於執法人員攜帶由其機構頒發的徽章，以使其權威合法化。令牌可以是實體的（如 USB 硬金鑰），也可以是數位的（電腦產生的訊息或數位簽名）。

## 實體令牌
通常在使用者登入過程中進行。使用者必須證明他們擁有其他人沒有的物品。他們可以透過輸入該物品顯示的程式碼、透過 USB 將物品連接到裝置、透過藍牙連接物品或其他幾種方法來證明這一點。
* 軟體令牌：指輸入傳送到裝置的密碼或訊息，以證明擁有該裝置。這通常為透過簡訊傳送到智慧型手機的程式碼。
* 硬體令牌：使用者直接連接到電腦或行動裝置以登入的硬體物品。

## Web 令牌
從伺服器傳送到用戶端並由用戶端臨時儲存的訊息。用戶端在傳送到伺服器的後續請求中包含令牌的複本，以確認用戶端的驗證狀態。
實體令牌驗證會在登入過程中驗證身分，而 Web 令牌則作為成功登入的結果發佈。

運作方式：
1. 用戶登錄：用戶提供其憑證（通常是用戶名和密碼）來登錄應用程式或網站。
2. 生成令牌：用戶成功登錄後，伺服器會進行身份驗證，並在驗證成功後為該用戶生成一個特定的令牌（token）。這個令牌通常是一個長隨機字串，代表了用戶的身份和權限。
3. 傳遞令牌：用戶獲得令牌後，在每次需要訪問受保護資源的請求中，都需要將這個令牌放置於請求中的特定位置，例如放在 HTTP 的 Authorization 標頭中。
4. 驗證令牌：伺服器接收到請求後，會驗證令牌的有效性和合法性。這涉及到檢查令牌的簽名、時效性和權限等信息。
5. 授權訪問：如果令牌是有效且合法的，伺服器將允許用戶訪問受保護的資源。

## JWT
在 Web 開發中，「Web 令牌」幾乎總是指JSON Web Token。JWT 是用於建立包含 JavaScript 物件標記法 (JSON) 資料的數位簽名 Web 令牌的標準。

傳遞方式會在 Header 中宣告`Authorization: Bearer <token>`。因為是透過 Header 傳遞且不依賴Cookie，也就不會有跨域請求的問題。
### JWT的組成
JWT的組合可以看成是三個JSON object，並且用`.`來做區隔，而這三個部分會各自進行編碼，組成一個JWT字串。例：`xxxxx.yyyyy.zzzzz`
#### Header
由兩個欄位組合：
1. alg：也就是token被加密的演算法，如HMAC、SHA256、RSA。
2. typ：也就是token的type，基本上就是JWT

範例：
```json=
{
    "alg": "HS256",
    "typ": "JWT"
}
```
然後進行Base64進行編碼。
#### Payload
這裡放的是聲明(Claim)內容，也就是用來放傳遞訊息的地方，在定義上有三種聲明：
1. Registered claims：
可以想成是標準公認的一些訊息建議你可以放，但並不強迫。例如：
iss (Issuer)： JWT 簽發者。
exp (Expiration Time)： JWT 的過期時間，過期時間必須大於簽發 JWT 時間。
sub (Subject)： JWT 所面向的用戶。
aud (Audience)：接收 JWT 的一方。
nbf (Not Before)：也就是定義擬發放 JWT 之後，的某段時間點前該 JWT 仍舊是不可用的。
iat (Issued At)： JWT 簽發時間。
jti (JWT Id)： JWT 的身分標示，每個 JWT 的 Id 都應該是不重複的，避免重複發放。
2. Public claims
可以想成是傳遞的欄位必須是跟上面 Registered claims 欄位不能衝突，然後可以向官方申請定義公開聲明，會進行審核等步驟，實務上在開發上是不太會用這部分的。

3. Private claims
這個就是發放JWT伺服器可以自定義的欄位的部分，例如實務上會放User Account、User Name、User Role等不敏感的數據。

因為該Payload傳遞的訊息最後也是透過Base64進行編碼，所以是可以被破解的，因此放使用者密碼會有安全性的問題。

範例：
```json=
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

通常都會放 iat、exp 等標準欄位，因為通常會需要檢查 JWT 發送時間及是否過期，以及還有使用者帳號，為了方便查詢使用者的一些數據。

#### Signature
由三大部分組成：
base64UrlEncode (header)
base64UrlEncode (payload)
secret
例如，如果要使用HMAC SHA256演算法：
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```
header 跟 payload 中間用`.`來串接， secret 是存放在伺服器端的秘密字串，最後將這三個部分串接並進行演算法加密。

secret 是要保存在伺服器端的，這個 secret 一旦外洩給客戶端，客戶端就可以自己產生 JWT ，並且透過該 JWT 存取資源，因此 secret 是永遠不該外流的。