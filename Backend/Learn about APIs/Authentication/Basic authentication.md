# Basic authentication
HTTP基本認證(HTTP Basic Authentication)，是一種簡單的HTTP請求認證方法。當client 端對server 發起請求的同時必須提供帳號(user-id)及密碼(password)讓server端驗證。

請求的HTTP Header 會包含 Authorization 欄位，形式如下： `Authorization: Basic <憑證>`，該憑證是帳號密碼(`<user-id>:<password>`)組成的base64 編碼。

## 安全性問題
基本認證並沒有為傳送憑證過程提供任何機密性的保護。僅僅使用 Base64 編碼並傳輸。因此，基本認證常常和 HTTPS 一起使用，以提供機密性。

```html=
GET /private/index.html HTTP/1.0
Host: localhost
```
```html=
HTTP/1.0 401 Authorization Required
Server: HTTPd/1.0
Date: Sat, 27 Nov 2004 10:18:15 GMT
WWW-Authenticate: Basic realm="Secure Area"
Content-Type: text/html
Content-Length: 311

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
 "http://www.w3.org/TR/1999/REC-html401-19991224/loose.dtd">
<HTML>
  <HEAD>
    <TITLE>Error</TITLE>
    <META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=ISO-8859-1">
  </HEAD>
  <BODY><H1>401 Unauthorized.</H1></BODY>
</HTML>
```
WWW-Authenticate 是 HTTP 標頭（header）中的一個指示，用於在需要進行身份驗證的情況下，通知客戶端必須提供指定的身份驗證方案。

一些較常見的身份驗證方案有（不區分大小寫）：`Basic`、`Digest`、`Negotiate`、`AWS4-HMAC-SHA256`。
