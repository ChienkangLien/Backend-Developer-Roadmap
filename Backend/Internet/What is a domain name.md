# What is a domain name?
域名是網路基礎設施的關鍵部分。它們為互聯網上可用的任何 Web 伺服器提供人類可讀的位址。

任何連接到 Internet 的電腦都可以透過公用IP 位址（IPv4 位址（例如192.0.2.172）或 IPv6 位址（例如2001:db8:8b73:0000:0000:8a2e:0370:1337））進行存取。

電腦可以輕鬆處理此類位址，但人們很難找出誰在運行伺服器或網站提供什麼服務。IP 位址很難記住，並且可能會隨著時間的推移而改變。

為了解決所有這些問題，我們使用人類可讀的地址（稱為網域）。
## 域名結構
網域具有由多個部分組成的簡單結構（可能只有一個部分、兩個、三個......），用點分隔並從右向左讀取：

![image](images/dnstructure.png)
每個部分都提供有關整個網域的特定資訊。

### TLD（頂級域名）。
TLD 告訴用戶域名背後服務的一般用途。最通用的 TLD ( .com、.org、.net) 不要求 Web 服務符合任何特定標準，但某些 TLD 執行更嚴格的政策，因此其目的更加明確。例如：
* 本地 TLD（例如.us、.fr、 或 ）.se可以要求以給定語言提供服務或在特定國家/地區託管 - 它們應該指示特定語言或國家/地區的資源。
* TLD.gov只允許政府部門使用。
* TLD.edu僅供教育和學術機構使用。

TLD 可以包含特殊字元和拉丁字元。TLD 的最大長度為 63 個字符，但大多數長度約為 2-3 個字符。
TLD 的完整清單由 ICANN 維護。

### 標籤（或組件）
標籤是 TLD 後面的內容。標籤是不區分大小寫的字符序列，長度為 1 到 63 個字符，僅包含字母A到Z、數字0到9以及“-”字符（可能不是標籤中的第一個或最後一個字符）。a、97、 和hello-strange-person-16-how-are-you都是有效標籤的範例。

位於 TLD 之前的標籤也稱為二級域名(SLD)。

一個網域可以有許多標籤（或組件）。一個網域並不強制也不需要有 3 個標籤。例如，www.inf.ed.ac.uk 是一個有效的網域。對於您控制的任何網域（例如 mozilla.org），您可以建立每個網域包含不同內容的“子網域”，例如 developer.mozilla.org、iot.mozilla.org 或 bugzilla.mozilla.org。
## 網域名稱和 URL 有什麼區別
統一資源定位符 (URL)，有時稱為網址，包含網站的網域名稱以及其他訊息，包括協定和路徑。例如，在 URL「https://cloudflare.com/learning/ 」中，「cloudflare.com」是域名，「https」是協議，「/learning/」是指向特定頁面的路徑網站。
## 管理域名
域名均由網域註冊機構管理，註冊機構將網域的使用權委託給註冊商。不能「購買網域」，而是向註冊商支付使用權費用。
### 尋找可用域名
域名註冊商大多數都提供“whois”服務，告訴您網域是否可用。
或者，如果您使用內建 shell 的系統，透過命令查找：
```
whois mozilla.org
```
這將輸出以下內容：
```
Domain Name:MOZILLA.ORG
Domain ID: D1409563-LROR
Creation Date: 1998-01-24T05:00:00Z
Updated Date: 2013-12-08T01:16:57Z
Registry Expiry Date: 2015-01-23T05:00:00Z
Sponsoring Registrar:MarkMonitor Inc. (R37-LROR)
Sponsoring Registrar IANA ID: 292
WHOIS Server:
Referral URL:
Domain Status: clientDeleteProhibited
Domain Status: clientTransferProhibited
Domain Status: clientUpdateProhibited
Registrant ID:mmr-33684
Registrant Name:DNS Admin
Registrant Organization:Mozilla Foundation
Registrant Street: 650 Castro St Ste 300
Registrant City:Mountain View
Registrant State/Province:CA
Registrant Postal Code:94041
Registrant Country:US
Registrant Phone:+1.6509030800
```
mozilla.org因為 Mozilla 基金會已經註冊了。如果可用的話會得到`NOT FOUND`
### DNS刷新
DNS資料庫儲存在全球每台DNS伺服器上，這些伺服器指的是一些特殊的伺服器，稱為「權威名稱伺服器」或「頂級DNS伺服器」。
向註冊商註冊域名後，幾個小時內，所有 DNS 伺服器都會收到您的 DNS 訊息，並在每個 DNS 資料庫中刷新該資訊。。