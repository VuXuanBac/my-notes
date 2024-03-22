---
  title:  Action Cable
  draft: true
  date: 2024-03-21
  tags:
    - realtime
    - ror
  description: Action Cable in Ruby on Rails
---

Action Cable giúp tích hợp **WebSockets** cho ứng dụng Rails, hỗ trợ các tính năng cần thời gian thực như gửi thông báo. Action Cable cung cấp cả một **client-side JS framework** và **server-side Ruby framework**

## WebSockets

WebSockets là một giao thức tầng ứng dụng dựa trên TCP, trong đó dữ liệu có thể truyền theo hai chiều trên cùng một kết nối (full-duplex). Ngay từ bước thiết kế, WebSockets đã có sự tương thích với HTTP, ngay cả cổng dịch vụ của nó cũng là 443 (**wss**) và/hoặc 80 (**ws**) và nó sử dụng giao thức HTTP để khởi tạo kết nối (Handshake).

Khác với HTTP, WebSockets là một Stateful Protocol, tức là kết nối giữa client và server sẽ được duy trì cho tới khi một trong hai bên ngắt kết nối. Vấn đề timeouts cũng không còn quá ảnh hưởng.

![WebSockets Protocol](assets/rails/websocket-protocol.png)

Nếu như trong HTTP, một bên thường phải gửi yêu cầu để nhận dữ liệu; kết nối hai chiều và stateful của WebSockets cho phép một bên chủ động gửi dữ liệu mà không cần có yêu cầu. Chính tính năng này giúp WebSockets phù hợp với các chức năng cần thời gian thực như chat, multiplayer games, financial tickers,...

## Action Cable Terminology

Tương tự kiến trúc của WebSockets, Action Cable bao gồm servers và clients (được gọi là **consumers**). Server được tạo bằng Ruby framework còn consumers được tạo bằng JS framework.

Server cung cấp cổng kết nối cho consumers thông qua các **channels**. Channel là đơn vị xử lý logic tương tự Controller trong mô hình MVC. Về phía của channels, consumers được gọi là **subcribers**, có thể đăng ký kết nối vào channel tương ứng. Kết nối từ channels và các subscribers được gọi là **subcriptions**.

Trong mô hình này, cả hai phía của kết nối đều có thể gửi dữ liệu, bên gửi được gọi là **publishers** và bên nhận được gọi là **subscribers**. Một publisher có thể gửi dữ liệu cho một nhóm các subscribers qua channels mà không cần quan tâm ai là subscribers.

**Broadcast** là hình thức gửi dữ liệu mà dữ liệu được gửi trực tiếp đến các **subcribers**.

## Components

Về phía Server, ta cần thiết lập các đối tượng sau:

1. Một Connection Class kế thừa từ `ActionCable::Connection::Base`, nó có nhiệm vụ xác thực và phân quyền cho consumer (clients của WebSocket protocol).

Mỗi yêu cầu kết nối từ consumers đến servers sẽ được xác thực trong Class này, nếu xác thực thành công, một _channel subsciption_ (connection) sẽ được tạo và gắn cho consumer đó.

Định danh cho _channel subsciption_ được xác định qua macro `identified_by` được định nghĩa trong Class này.

2. Các Channel Class kế thừa từ `ActionCable::Channel::Base`

Khi consumer được chấp nhận sẽ trở thành **subscribers** của channel, và method **`subscribed`** trong Channel Class tương ứng sẽ được gọi.

Về phía Client, ta cần thiết lập các đối tượng sau:

1. Một Consumer (client của WebSocket), JS framework cung cấp function `createConsumer` (từ thư viện `@rails/actioncable`).

Đối số của function này là URL của WebSocket server, mặc định là `/cable`

2. Dùng Consumer vừa tạo để đăng ký vào một/nhiều Channel đã tạo trên Server (hoặc một Channel và đăng ký nhiều lần)
