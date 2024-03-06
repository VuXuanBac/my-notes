---
title: Rack and Puma
draft: false
tags:
  - ror
  - system
---

# Architecture

![Work Flow](assets/rails/web_workflow.png)

**Puma** là một Application Server, Web Server (ngoài ra còn có Unicorn, Passenger, WEBrick,... - cũng tương tự với Tomcat của Java)

Một Application Server nhận vào một HTTP Request, xử lý và parse nó để ứng dụng có thể sử dụng. Tương tự, nó cũng nhận Response từ ứng dụng và parse về HTTP Response.

**Rack** là một giao diện phát triển Web trong Ruby, nó nhận vào các Requests đã được Parsed từ Application Server, và xử lý qua các Middleware trước khi chuyển cho ứng dụng (Rails, Sinatra,...). 

Rack hỗ trợ:
- Web Servers: Agoo, Puma, Falcon, Unicorn,...
- Web Frameworks: Sinatra, Ruby on Rails, Hanami,..
- Middlewares: Logger, Sendfile, Static,...

Rails Dispatcher: Nạp lại mã nguồn mỗi khi có request gửi tới server

Web console: Màn hình console trực tiếp trên Website khi có lỗi
- Mặc định chỉ xuất hiện khi cùng IP với server
- Có thể cấu hình whitelists trong _config/environments/development.rb_. Ex:    
    - Anybody: `config.web_console.whitelisted_ips = %w( 0.0.0.0/0 ::/0 )`

# Engines

**Engines** là các ứng dụng nhỏ cung cấp chức năng cho ứng dụng hiện tại (Host Application). VD: Devise, Thredded, Spree,...

Sử dụng Engine trong ứng dụng:
- Trước hết là thêm nó vào Gemfile, với Option `path` xác định đường dẫn tới Engine đó. 
    - Việc này giúp Engine được nạp khi nạp ứng dụng.
- Các chức năng của Engine có thể truy xuất qua Routes, và để chỉ định có một Engine tại một đường dẫn, ta khai báo **mount** bên trong _config/routes.rb_
    - VD: `mount Sidekiq::Web => "/jobs"`
    - **Devise** sử dụng Helper `devise_for` để thay thế `mount`