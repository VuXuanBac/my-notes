---
title: PostgreSQL
draft: true
date: 2024-03-{3:13}
tags:
  - database
description: PostgreSQL
aliases:
---

# Roles

PostgreSQL quản lý quyền truy cập vào DBs theo các **Roles**, các roles có thể sở hữu một DB object (table, function,...), có thể trao quyền của mình cho các roles khác.

Role mặc định được tạo cùng với PostgreSQL là **`postgres`**, role này mặc định là **superuser** với các quyền: Tạo roles, Tạo DB,... Role này cũng tương ứng là một user của hệ thống.

Để tương tác với PostgreSQL, trước tiên cần sử dụng role này:

```cmd
# Access PostgreSQL terminal
sudo -u postgre psql [-U <role>]
```

Các thao tác với roles qua SQL

```sql
CREATE ROLE <name>

DROP ROLE <name>

ALTER ROLE <name>
```

PostgreSQL cung cấp lệnh `\du` để hiển thị danh sách roles.

Riêng `CREATE ROLE` và `DROP ROLE` có thể gọi nhanh (không cần truy câpj vào Terminal) qua lệnh `createuser` và `dropuser`.

## Role Attributes

Mỗi role có các [thuộc tính](https://www.postgresql.org/docs/current/role-attributes.html), ví dụ:

- Password: Mật khẩu dùng để kết nối vào tài khoản này.
  ```sql
  CREATE/ALTER USER PASSWORD '<password>';
  ```
- SuperUser: Tài khoản này được cấp mọi quyền
  ```sql
  CREATE/ALTER USER SUPERUSER;
  ```
- Database Creation: Tài khoản này có thể tạo mới các DBs
  ```sql
  CREATE/ALTER USER CREATEDB;
  ```
- Login: Có thể kết nối tới DB qua role này
  ```sql
  CREATE/ALTER USER LOGIN;
  ```

# Connect to DB

Khi muốn kết nối tới DB Server, ứng dụng cần xác định username (role name) và các thông tin khác để xác thực.

Các cấu hình về cơ chế xác thực được lưu tại tệp **pg_hba.conf**. Vị trí của tệp này có thể xác định từ SQL command: `SHOW hba_file;`

```yaml
# DO NOT DISABLE!
# If you change this first entry you will need to make sure that the
# database superuser can access the database using some other method.
# Noninteractive access to all databases is required during automatic
# maintenance (custom daily cronjobs, replication, and similar tasks).
#
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
```

Các records trong tệp này xác định white lists cho các kết nối vào PostgreSQL DBs, mỗi record gồm 5 trường:

- Connection Type: Kiểu kết nối tới DB
  - **`local`**: Kết nối trực tiếp qua Socket của máy cục bộ.
  - **`host`**: Kết nối qua TCP/IP.
- Database: DB cho phép kết nối tới.
- User: Role
- Address: Địa chỉ IP/Hostname nếu kiểu kết nối là `host`
- Authentication Method: Cơ chế xác thực
  - `peer`: [Chỉ dùng với `local`] Kết nối được chấp nhận nếu role trùng với tài khoản người dùng của hệ thống.
  - `md5`: Xác thực qua mật khẩu
