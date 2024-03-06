---
title: Grape API
draft: false
tags:
  - ror
  - gem
  - api
---

Grape là một API Framework cho Ruby. Nó có thể chạy độc lập trên [Rack](../other/rack_puma.md) hoặc mở rộng chức năng cho Rails/Sinatra Framework.

Cấu hình:

```cmd
bundle add grape
```

Grape APIs là các ứng dụng chạy trên Rack, ta cần tạo các Classes kế thừa từ `Grape::API` để tạo một ứng dụng cung cấp APIs.
- Base Class này triển khai một method `call`, là interface point giữa Rack và các ứng dụng chạy trên nó (ở đây là Grape). Method này nhận một request từ Rack và phản hồi về một mảng gồm **status**, **headers** và **body**