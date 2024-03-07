---
title: Docker
draft: true
date: 2024-03-07
tags:
  - system
  - docker
---

Docker là một nền tảng giúp loại bỏ sự phụ thuộc của ứng dụng vào nền tảng bên dưới.

Docker đóng gói và chạy ứng dụng trên một môi trường độc lập gọi là **container**. Sự độc lập này cho phép có thể chạy nhiều containers cùng trên một nền tảng phần cứng - **host**.

![Docker Architecture](assets/docker/architecture.png)

Docker sử dụng kiến trúc Client-Server:
- **Deamon** (`dockerd`): 
    - Quản lý các đối tượng: **images, containers, networks, volumes**
    - Quản lý việc **build**, **run** và **phân phối** các containers. 
    - Giao tiếp với Deamons khác
    - Lắng nghe và xử lý các Request từ Client.
- **Client** (`docker`): Giao diện giúp người dùng tương tác với Docker bằng việc gửi requests đến một hay nhiều Deamons
- **Registry**: Nơi chứa các **images**
  - **DockerHub** là một public registry, ở đó có thể sử dụng các images mà không yêu cầu xác thực hay cấp quyền.

# Docker objects

## Images

Image là bản hướng dẫn để tạo môi trường chạy của ứng dụng. Các **images** thường được tạo dựa trên các image cơ sở và chỉ thực hiện một số cấu hình riêng.

Khi build một image, ta được một container. Các lệnh hướng dẫn build một image được ghi trong **Dockerfile**.

Mỗi chỉ thị trong **Dockerfile** sẽ tương ứng tạo một Layer trên image. Điều này giúp:
- Download (Pull) các Layers đồng thời.
- Nếu có sự thay đổi chỉ cần nạp lại Layers có sự thay đổi.

## Container

Container là phiên bản chạy được của image. Các containers độc lập với nhau và độc lập với host, tuy nhiên, mức độ độc lập có thể cấu hình.

Container là một process được chạy độc lập với các processes khác.

# Command List

| Command  | Description  | Options |
|---|---|---|
| `pull <name>` | Download Image từ Registry | |
| `run <name>` | Chạy một Container từ Image (Download nếu chưa có) với các cấu hình ban đầu  | `-d`: Detach mode, chạy trong Background<br>`-pHOST:CONTAINER`: Binding từ một Host Port sang một Port trong Container<br>`--name <name>`: Đặt tên gợi nhớ cho Container |
| `stop <id>` | Dừng một Container | |
| `start <id>` | Chạy một Container đã được tạo từ lệnh `run` | |
| `rm <id>` | Xóa Container | `-f`: Dừng Container trước khi xóa |
| `images`  | Danh sách Images được lưu cục bộ  | |
| `ps` | Danh sách Containers đang chạy | `-a`: Bao gồm cả Containers đã dừng |
| `logs <id>` | Hiển thị Log trả ra từ Container | |
| `exec <id> <command>` | Chạy command trong Container


# Example

Thêm file **Dockerfile** vào project để hướng dẫn build image. VD: Một ứng dụng NodeJS có thể tạo một image như sau

```cmd
# syntax=docker/dockerfile:1

FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```

Sau đó chạy lệnh sau để tạo build image.

```cmd
docker build -t <name> path/to/Dockerfile
```

Với `<name>` là tên tham chiếu tới image.

Để chạy một container, ta sử dụng lệnh

```cmd
docker run -dp <HOST>:<CONTAINER> <name>
```

Option `-d` thực hiện chạy container trong Background. 

Option `-p` dùng để ánh xạ một cổng trong Host với một cổng của container. 
- Đối số của nó có dạng `<HOST>:<CONTAINER>` với `<HOST>` là địa chỉ của Host (`127.0.0.1:3000`) và `<CONTAINER>` là cổng của container.
- Ánh xạ này giúp truy cập cổng trong container qua host.

# Dockerfile

Dockerfile là một tệp text chứa các chỉ thị build image. Docker sẽ chạy các lệnh trong Dockerfile theo thứ tự.

Các Dockerfile luôn phải bắt đầu bằng một lệnh `FROM` để xác định base image mà image hiện tại kế thừa.

Ta có thể thiết lập các biến môi trường qua lệnh `ENV`, từ đó, có thể truy xuất qua `$<name>` hoặc `${<name>}`

## WORKDIR

`WORKDIR` dùng để xác định đường dẫn tới thư mục làm việc trong container. Nếu thư mục chưa tồn tại, Docker sẽ tạo mới

## RUN, CMD and ENTRYPOINT

Có 3 lệnh để thực thi các lệnh là `RUN`, `CMD` và `ENTRYPOINT`. 3 lệnh này có chung cú pháp:

```cmd
COMMAND ["executable", "param1", "param2"] # Dạng Exec
COMMAND command param1 param2 # Dạng shell
```

Dạng exec truyền vào một mảng JSON (sử dụng `"`), các phần tử là đường dẫn đến tệp thực thi và các đối số và flags. Đường dẫn cần phải Backslash

Dạng shell luôn thực thi lệnh trên shell. Đường dẫn đến shell được xác định qua `SHELL`. Shell mặc định là `/bin/sh -c`

Lệnh `RUN` dùng để thực thi các lệnh tạo Layer mới trên image.

Lệnh `ENTRYPOINT` để xác định tệp thực thi dùng để chạy các lệnh, mặc định là `/bin/sh -c`. Các lệnh truyền cho `CMD` sẽ được chạy trên tệp thực thi này

Lệnh `CMD` dùng để xác định các lệnh sẽ được chạy cùng với container. 
- Toàn bộ Dockerfile **chỉ có một** lệnh `CMD`, dùng để xác định các đối số mặc định cho `ENTRYPOINT`.
- Các đối số này có thể bị ghi đè khi chạy container `docker run`

Dockerfile cần xác định ít nhất một trong hai lệnh `CMD` hoặc `ENTRYPOINT`

![CMD and ENTRYPOINT](assets/docker/entrypoint.png)

## EXPOSE

`EXPOSE` dùng để chỉ thị một/nhiều cổng mà container mở sẵn, với giao thức xác định. Cú pháp

`EXPOSE <port>/<protocol>`

Lệnh này giống như bản mô tả để việc chạy container và tạo image thống nhất với nhau.

## ADD vs COPY

`ADD` và `COPY` dùng để sao chép tệp/thư mục từ đường dẫn **src** vào đường dẫn **dest** tới hệ thống tệp của image.
- Có thể xác định nhiều **src**, là các đường dẫn tương đối so với thư mục thực hiện build, tuy nhiên không thể truy cập các thư mục cha của nó.
- Các đường dẫn trong **dest** có thể là tuyệt đối hoặc tương đối so với `WORKDIR`

Ngoài ra, `ADD` có thêm hai chức năng:
- Tự động giải nén **gzip, bzip2, xz** thành thư mục
- Download tệp từ URL

# Persist the DB

Các containers chạy độc lập với nhau, sẽ luôn sử dụng Filesystem được khởi tạo cùng với chúng. Như vậy, mỗi lần chạy một container instance.

Do đó, để lưu trữ lâu dài dữ liệu trên container, ta cần ánh xạ nó sang ổ cứng của máy host. Cơ chế này được cung cấp từ [**Volume**](https://docs.docker.com/storage/volumes/).
- Thay đổi trên tệp có thể đọc từ host.
- Các containers có thể mount tới thư mục Volume này.

![Docker Volume](assets/docker/volume.png)

Ta có thể tạo Volume với lệnh

```cmd
docker volume create <volume-name>
```

Để xem thông tin về Volume vừa tạo, bao gồm vị trí lưu Volume trên ổ cứng, sử dụng lệnh

```cmd
docker volume inspec <volume-name>
```

Để mount một container với một Volume, sử dụng lệnh

```cmd
docker run ... --mount type=volume,src=<volume-name>,target=<container-directory>
```

Ta có thể tạo và mount đồng thời Volume khi chạy container qua option `-v`

```cmd
docker run ... -v <volume-name>:<target>
```

# Bind Mount

Bind Mount là một cơ chế giúp đồng bộ thay đổi trên một thư mục trên host với hệ thống tệp trong container.

Cơ chế này rất hữu dụng trong việc phát triển ứng dụng, giúp build và chạy ứng dụng từ container mà không cần các công cụ phát triển trên host.

# Multi-container App

Việc phát triển ứng dụng nhiều khi cần chạy nhiều containers cho các dịch vụ khác nhau: DB server, web server,...

Để các containers trao đổi thông tin với nhau, ta cần kết nối chúng qua một **Network**.

Ta tạo một Network qua

```cmd
docker network create <network-name>
```

và có thể gắn một container vào một Network qua

```cmd
docker run ... --network <network-name> --network-alias 
```

## Docker Compose

Việc tạo và chạy thủ công các containers cần giao tiếp với nhau khiến khó quản lý. Ta có thể sử dụng Docker Compose để khai báo các lệnh này vào một file YAML.

Trong thư mục ứng dụng, tạo tệp **`compose.yaml`**

VD: Cần chạy một ứng dụng NodeJS và MySQL

```cmd
<!-- NodeJS -->
docker run -dp 127.0.0.1:3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:18-alpine \
  sh -c "yarn install && yarn run dev"

<!-- MySQL -->
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:8.0
```

Tương ứng với chúng là file **`compose.yaml`** như sau

```yaml
services:
  app: # --name
    image: node:18-alpine # image-name
    command: sh -c "yarn install && yarn run dev" # command
    ports: # -p
      - 127.0.0.1:3000:3000
    working_dir: /app # -w
    volumes: # -v
      - ./:/app
    environment: # -e
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
  db: # name for container 
    image: mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

Với **compose.yaml**, ta có thể chạy các containers như sau:

```cmd
docker compose up -d
```

Để dừng các containers và xóa network, sử dụng

```cmd
docker compose down [--volumes]
```

# Example

Ví dụ này cấu hình chạy một ứng dụng RoR với PostgreSQL DB trên Docker