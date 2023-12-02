# SOAP
Simple Object Access Protocol 簡單物件存取協定，是交換資料的一種協定規範，遵從XML格式執行資料互換。

但它的複雜性和相對較為冗長的訊息格式，使得它在當今 Web API 的使用上逐漸被更輕量、靈活的協定（如 RESTful API）所取代。
## 傳輸方式
使用網際網路應用層協定作為其傳輸協定。 SMTP 以及 HTTP 協定都可以，但是由於 HTTP 在如今的網際網路結構中工作得很好，特別是在網路防火牆下仍然正常工作，所以被廣泛採納。SOAP亦可以在HTTPS上傳輸。

## 語法規則
* 必須使用SOAP Envelope命名空間
* 必須使用SOAP Encoding命名空間
* 不能包含DTD參照
* 不能包含XML處理指令


請求sample：
```xml=
POST /InStock HTTP/1.1
Host: www.example.org
Content-Type: application/soap+xml; charset=utf-8
Content-Length: nnn

<?xml version="1.0"?>

<soap:Envelope
xmlns:soap="http://www.w3.org/2003/05/soap-envelope/"
soap:encodingStyle="http://www.w3.org/2003/05/soap-encoding">

<soap:Body xmlns:m="http://www.example.org/stock">
  <m:GetStockPrice>
    <m:StockName>IBM</m:StockName>
  </m:GetStockPrice>
</soap:Body>

</soap:Envelope>
```

回應sample：
```xml=
HTTP/1.1 200 OK
Content-Type: application/soap+xml; charset=utf-8
Content-Length: nnn

<?xml version="1.0"?>

<soap:Envelope
xmlns:soap="http://www.w3.org/2003/05/soap-envelope/"
soap:encodingStyle="http://www.w3.org/2003/05/soap-encoding">

<soap:Body xmlns:m="http://www.example.org/stock">
  <m:GetStockPriceResponse>
    <m:Price>34.5</m:Price>
  </m:GetStockPriceResponse>
</soap:Body>

</soap:Envelope>
```

更多內容請見 https://www.w3schools.com/xml/xml_soap.asp