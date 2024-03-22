---
  title:  Realtime with Websocket and Webhook
  draft: false
  date: 2024-03-21
  tags:
    - til
    - service
    - realtime
  description: What is WebSockets? What is WebHooks? When to use them?
---

> 🐆 Điều gì đứng phía sau các tính năng cần cập nhật thời gian thực trong các ứng dụng ? 🥰

🎈 Những tính năng như chat, notifications,... quá phổ biến trong các ứng dụng, và để phát triển các tính năng này, ta cần các kỹ thuật xử lý thời gian thực - **realtime**, tức là khi có dữ liệu mới, dữ liệu này được gửi về cho các bên liên quan trong thời gian đủ nhỏ để không ảnh hưởng đến trải nghiệm người dùng.

🔎 Dưới đây là một số giải pháp cho các tính năng này

> [!note]
> Dưới đây mình sử dụng **`server`** và _`client`_ theo ý nghĩa của **`bên cung cấp dịch vụ/dữ liệu`** và _`bên yêu cầu dịch vụ/dữ liệu`_
> Nghĩa là: _một máy tính có thể là server trong ngữ cảnh này, có thể là client trong ngữ cảnh khác_

### 🏓 Polling

Một trong những cách sơ khai mà ai cũng có thể nghĩ đến là **định kỳ gửi yêu cầu cập nhật dữ liệu mới** (_short polling_). Rõ ràng hạn chế của cách xử lý này làm **tăng tải cho server**.

![Short Polling](assets/til/realtime-short-polling.png)

Cơ chế Polling thứ hai là **server chỉ gửi về dữ liệu khi có** (_long polling_)

![Long Polling](assets/til/realtime-long-polling.png)

Cách này giảm lượng yêu cầu gửi đến server, tuy nhiên, server lại cần duy trì kết nối tới client trong suốt thời gian chờ có dữ liệu. Hạn chế của cách này chính là **tiêu tốn tài nguyên trên server** 🔥, vì mỗi kết nối sẽ tương ứng một phần bộ nhớ bị chiếm dụng (tương ứng với một process hoặc thread), và thường số kết nối cho phép cho các servers chỉ có giới hạn, khi đạt đến giới hạn đó, các kết nối đến sau sẽ không thể được phục vụ 🚧.

Có thể thấy, điểm yếu của các giải pháp trên là **cơ chế dừng chờ - đồng bộ**. Những giải pháp dưới đây sẽ cho thấy một góc nhìn khác.

### 📭 WebSockets

WebSockets triển khai cơ chế giao tiếp theo cách khác (cùng trên nền của giao thức TCP), ở đó kết nối giữa client và server là **full-duplex** (có thể gửi và nhận đồng thời trên cùng một kênh truyền).

> [!info]
> Kết nối HTTP gọi là **half-duplex** - kênh truyền có thể truyền hai chiều nhưng tại một thời điểm, các bên chỉ có thể gửi hoặc nhận.

![WebSockets Flow](assets/til/realtime-websocket-flow.png)

Sau bước bắt tay 🤝 thông qua giao thức HTTP (HTTP Upgrade - status code 101), kết nối giữa client và server được duy trì, mọi dữ liệu trao đổi diễn ra trên kết nối đã thiết lập và kết nối chỉ đóng khi một trong hai bên chủ động kết thúc phiên.

Mặc dù với điểm mạnh là độ trễ thấp và kết nối có thể dùng để trao đổi hai chiều đồng thời, WebSockets có một số hạn chế sau:

- Kết nối luôn duy trì đòi hỏi thiết bị client phải dành một phần bộ nhớ để sử dụng, và cũng **tiêu tốn năng lượng**, điều này không phù hợp với các thiết bị di động.
- WebSockets là giao thức trên nền TCP, mặc dù hướng cho dịch vụ Web nhưng **một số chức năng của HTTP (như compression) không có sẵn ở giao thức này**.

### 🫸 Server-sent Events (SSE)

Khác với WebSocket, SSE chỉ duy trì luồng truyền dữ liệu theo **một chiều, từ server về client** (_server push_)

![SSE Flow](assets/til/realtime-sse-flow.png)

Cụ thể, server sinh sẵn một số DOM Events, sau đó client đăng ký vào một số Events đó để nhận dữ liệu cập nhật khi Event được kích hoạt. Server sử dụng XHR streaming để truyền dữ liệu về client.

Với đặc điểm như vậy, có thể thấy một số hạn chế của giải pháp này:

- Dữ liệu chỉ truyền một chiều từ server về client nên giải pháp này khá kén chọn ứng dụng, thông thường, các ứng dụng chỉ đọc kiểu như **Stock Tickers, Feed Update, Notifications** là phù hợp.
- Các Browsers [giới hạn số kết nối SSE đồng thời đến server (domain)](https://stackoverflow.com/questions/18584525/server-sent-events-and-browser-limits?trk=article-ssr-frontend-pulse_little-text-block)
- SSE vẫn cần duy trì kết nối giữa client và server.

### 🪝 WebHooks

Nếu như với SSE, kết nối giữa client và server cần duy trì để luôn sẵn sàng cho server gửi dữ liệu thì WebHook cho phép kết nối giữa hai bên có thể đóng lại với mỗi phiên trao đổi dữ liệu.

> WebHook có một số tên gọi khác là **Web Callback**, **HTTP Push API**.

Giống với WebSockets và SSE, WebHooks cũng là một phương pháp **hướng sự kiện**, cho phép xử lý bất đồng bộ các yêu cầu dịch vụ. _Khi sự kiện đã đăng ký trước được kích hoạt 🧨, một/chuỗi các hành động gắn với nó cũng sẽ được thực thi_.

> WebHook đôi khi còn được gọi là **Reverse API**, dữ liệu được gửi từ server về client thông qua một API được mở tại client.

Như vậy, để nhận được dữ liệu từ server, client cần mở một API và gửi địa chỉ của API đó cho server (_consuming a webhook_). Khi một sự kiện xảy ra trên server, nó gửi dữ liệu đến client qua URL đó.

Với cách thức hoạt động này, WebHook thường sử dụng trong trường hợp server của một ứng dụng cần sử dụng dữ liệu từ bên ngoài, ví dụ tích hợp dịch vụ thanh toán (bên thanh toán sẽ gửi về ứng dụng thông tin kết quả thanh toán)

![WebHook with Shopify](assets/til/realtime-webhook-example.png)

WebHooks cho phép các servers đồng bộ dữ liệu với nhau

### 📬 Tóm lại

So sánh giữa WebSockets, SSE và WebHooks

![WebSockets vs SSE vs WebHooks](assets/til/realtime-websockets-sse-webhooks.png)

### 🪝Tham khảo

- [Sandeep Rastogi Post](https://www.linkedin.com/pulse/real-time-web-application-whatre-different-ways-play-livereal)
- [Anghel Leonard Twitte](https://twitter.com/anghelleonard/status/750976309292040192)
- [5 Giao thức hướng sự kiện](https://nordicapis.com/5-protocols-for-event-driven-api-architectures/)

### 📚 Đọc thêm

- [Một số ứng dụng của WebSocket](https://ably.com/topic/what-are-websockets-used-for)
- [Một số ứng dụng của WebHook](https://blog.iron.io/7-reasons-webhooks-are-magic/)
- [Cách để tạo một WebHook Provider trong Rails với Sidekiq](https://keygen.sh/blog/how-to-build-a-webhook-system-in-rails-using-sidekiq/)
