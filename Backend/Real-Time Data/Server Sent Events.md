# Server Sent Events
Server-Sent Events（SSE）是一種建立在基於HTTP的單向持續連接上的瀏覽器和伺服器之間的通訊機制。
是一個已經被 W3C 納入 HTML5 標準的 API，它允許伺服器向客戶端推送數據，這樣客戶端就可以接收來自伺服器的更新。

傳統的網頁架構下，如果瀏覽器要持續接收來自於伺服器端的新資料時，通常都是透過 Polling、Long-Polling 或 Streaming 等方式來達成，而後來出現的 WebSocket 徹底解決了這個問題，不過除此之外，在 HTML5 標準中還有一個 Server-Sent Events 也可以處理這類型的問題。

SSE 鮮為人知的原因主要是由於 WebSocket 功能實在太強大了，WebSocket 提供了雙向（bi-directional）且全雙工（full-duplex）的優異傳輸能力，乍看之下 SSE 可以做到的事情，WebSocket 可以做得更好。

由於 WebSocket 所使用的是雙向全雙工的連線，所以需要特別支援 WebSocket 協定的網頁伺服器軟體才能讓它正常運作，而 SSE 則是一種架構在傳統的 HTTP 協定之上的傳輸方式，也就是說你可以在不需要加裝任何特別的通訊協定或伺服器軟體即可直接使用 SSE，另外 SSE 也有一些 WebSocket 所沒有的特性，例如自動重新連線、事件 ID 與傳送任意的事件等，這些都是 SSE 才有的優點。

特點包括：
* 單向持續連接： SSE 允許伺服器向客戶端建立一條持久的單向連接，這樣伺服器就可以定期推送數據給客戶端。
* 基於標準的HTTP協議： SSE 建立在標準的 HTTP 或 HTTPS 協議上，使用標準的 HTTP 請求和響應，因此它與大多數 Web 伺服器和瀏覽器兼容。
* 簡單易用： SSE使用簡單明瞭的 API，通常由 JavaScript 提供支持，並使用 EventSource 物件來建立與伺服器的連接。
* 文本數據格式： SSE 主要傳遞的是文本數據，通常是以純文本或 JSON 格式的事件流。

工作原理：
* 客戶端連接伺服器： 客戶端使用 JavaScript 中的 EventSource 物件建立到伺服器的 HTTP 連接。
* 伺服器推送數據： 一旦連接建立，伺服器可以透過這個已建立的連接定期推送事件流給客戶端。
* 客戶端接收事件： 客戶端通過監聽事件的方式接收來自伺服器的數據，並根據需要對其進行處理。

## 伺服器端（使用Java Spring Boot）
```java=
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

@RestController
public class SSEController {

    @GetMapping("/events")
    public void getEvents(javax.servlet.http.HttpServletResponse response) {
        response.setContentType("text/event-stream");
        response.setCharacterEncoding("UTF-8");

        ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
        executor.scheduleAtFixedRate(() -> {
            try {
                response.getWriter().write("data: This is a server-sent event\n\n");
                response.getWriter().flush();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, 0, 3, TimeUnit.SECONDS); // 每3秒推送一次事件

    }
}
```
當客戶端訪問 /events 端點時，它會建立一個事件流並每3秒推送一個事件。
## 客戶端（使用JavaScript）
```javascript=
const eventSource = new EventSource('/events');

eventSource.onmessage = function(event) {
    console.log('Received event: ', event.data);
    // 在這裡處理接收到的事件數據
};

eventSource.onerror = function(error) {
    console.error('EventSource failed:', error);
    eventSource.close();
};
```
使用 JavaScript 的 EventSource 物件來建立到 /events 端點的連接。當伺服器端推送事件時，onmessage 事件會觸發並處理接收到的事件數據。