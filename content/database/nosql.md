---
title: NoSQL
draft: false
tags:
    - database
---

No-SQL (Non-Relational) là một hướng thiết kế DB tập trung vào **lưu trữ** và **lấy** dữ liệu hơn là **quan hệ** về dữ liệu.

Khác với Relational DB chứa dữ liệu theo cấu trúc bảng thì NoSQL DB sử dụng một cấu trúc dữ liệu (VD: Key-value, wide column, graph, document (XML, JSON,...)). NoSQL DB có ưu điểm về tính mở rộng, dễ quản lý. Do đó được ứng dụng nhiều trong Big Data và Real-time web.

Phân loại theo Data Model:
- **Key-value Store** [Hash]: Redis, Memcached, Velocity,
- **Document Store**: [Cấu trúc tệp như XML, JSON,...] MongoDB, ArangoDB
- **Graph**: ArangoDB, IBM Db2, Sparksee
- **Column Store**: Cassandra, Bigtable, Amazon DynamoDB

# Redis

Redis (Remote Dictionary Server) là một hệ thống lưu trữ dữ liệu dưới dạng **key-value**, một **in-memory** **data structure store** hướng đến **performance** hơn là **persistence**.

Redis có thể được sử dụng cho Database, Cache, Streaming Engine, Message Broker,....  và hỗ trợ nhiều kiểu dữ liệu như Hash, List, Sorted Set,...

Redis sử dụng Memory (RAM) để lưu trữ và phục vụ dữ liệu. Redis cũng cung cấp cơ chế Persistence để lưu trữ dữ liệu lâu dài.

Chính vì dữ liệu được lưu trong RAM nên ưu điểm (so với các giải pháp Disked-base Storage như SQL DB) của Redis là **tốc độ** nhưng hạn chế là **kích thước bộ nhớ**.

## Persistence

Redis cung cấp một số mức persistence cho dữ liệu.
- **RDB (Redis Database)**: Cứ sau một khoảng thời gian, thực hiện lưu một Snapshot cho dữ liệu hiện tại
- **AOF (Append Only File)**: Log lại các thao tác ghi. Như vậy, khi Redis Server khởi động lại, dữ liệu có thể khôi phục dựa trên Log.
    - Kích thước Log có thể lớn hơn cả dữ liệu thực sự (RDB)
- **None**: Không lưu trữ lâu dài
- **RDB + AOF**

# ElasticSearch

Elasticsearch là một search engine phân tán, cung cấp giao diện RESTful API để các ứng dụng tương tác.

ES là một **document-based noSQL DB**, hướng đến tối ưu cho tốc độ và sự liên quan của kết quả tìm kiếm bằng việc đánh **inverted index**.

> **inverted index** chỉ việc tạo ánh xạ cho nội dung (các từ, số) tới vị trí của nó trong một bảng, tài liệu.

