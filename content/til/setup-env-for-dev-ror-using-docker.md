---
  title: Setup environment for developing RoR using Docker
  draft: false
  date: 2024-03-09
  tags:
    - ror
    - til
    - system
  description: Want an environment for developing RoR with Docker?
  aliases:
---

#### ⚡ Vấn đề

> 🪿 **Hôm nay mình đã thử kết hợp cả Docker và Virtual Machine để giảm thiểu lượng dependencies phải cài đặt vào máy Host cho dự án.** 😌

🎈Mình đang có một dự án cá nhân với RoR và PostgreSQL (sau có thể sử dụng cả Redis nữa), và mình không muốn phải cài mọi thứ vào máy Host (Windows).

🔎 Do đó, mình đã thử kết hợp:

- Virtual Machine (Ubuntu) cho RoR
- Docker cho Postgres (và Redis)

Không có lý do gì cho sự kết hợp này cả 😁, và như thế mình mới có một bài TIL... 🤣.

Hãy bắt đầu thiết lập môi trường

#### 🚚 Virtual Machine - RoR

💰 Để quản lý Ruby version, mình sử dụng **rbenv**

```cmd
sudo apt update

# Install Ruby dependencies
sudo apt install git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libreadline-dev libncurses5-dev libffi-dev libgdbm-dev

# Fetch script rbenv-installer and Run it with `bash`
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash

# Add `rbenv` to $PATH
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc

# Add `rbenv init` to `~/.bashrc` to run it automatically
echo 'eval "$(rbenv init -)"' >> ~/.bashrc

# Apply change on `~/.bashrc` right on current shell session
source ~/.bashrc

# rbenv is available now

# List all available versions of Ruby
rbenv install -l

# Install Ruby
rbenv install 3.3.0

# Set this version as global
rbenv global 3.3.0

# Ruby is available now
```

💎 Về cấu hình cho **gem**

```cmd
# Config Gem to Do not generate document for installed gem
echo "gem: --no-document" > ~/.gemrc

# Bundler for manage gem dependencies
gem install bundler
```

🛤 Tải Rails qua Gem

```cmd
gem install rails
```

🇻🇳 Ngoài ra, mình cũng tải thêm bộ hỗ trợ gõ Tiếng Việt cho Ubuntu, sử dụng **ibus-unikey**

```cmd
sudo apt install ibus-unikey

ibus restart
```

Sau đó vào Settings -> Region & Language -> Input Sources để thêm một ngôn ngữ. Lựa chọn **Vietnamese (Unikey)**

#### 🐳 Docker - PostgreSQL

📁 Trên máy Host (Windows), mình tạo thư mục để chứa các tệp liên quan cần cho Project

Tạo một Docker Compose file trong thư mục này

```yaml
# <project-name>/compose.yaml
version: "3"

services:
  postgres:
    image: postgres
    environment:
      - POSTGRES_USER=<username-to-connect>
      - POSTGRES_PASSWORD=<password-to-connect>
    ports:
      - "5432:5432"
    restart: always
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

🔆 **Giải thích một chút**

- Sử dụng Image `postgres:latest`
- Tạo hai biến môi trường để tạo sẵn một Role/User cho PostgreSQL.
- Postgres kết nối qua cổng **5432** nên cần mở một cổng trên Host (ở đây là 5432) và ánh xạ nó tới cổng này.
- Tạo một Volume `pgdata` để chứa dữ liệu DB sinh ra từ Postgres.

#### 🔗 Connect them

✨Đây là phần thú vị nhất của bài này - **Kết nối Docker với Virtual Machine**

##### 🪐 Virtual Machine Networking

Trước hết, mình sẽ nói qua một số cách kết nối từ Virtual Machine ra môi trường bên ngoài.

Các Virtual Machines có thể mở một số cổng kết nối - **Adapter** (VD: VirtualBox cho phép mở 4 cổng - Xem mục _Settings/Network_), các adapters này có thể hoạt động trong một trong các modes sau:

- 🫥 **NAT**: Tạo một Router (Application) giữa VM và Host. Router này có nhiệm vụ
  - NAT IP cho VM (VM thường có IP dạng 10.0.x.y)
  - Xử lý gói tin và trao đổi ra mạng bên ngoài giống như cách một ứng dụng trên Host.
- 🌐**Bridge**: Sử dụng cổng mạng trực tiếp của Host để kết nối.
- 📫 **Internal**: Tạo một mạng dựa trên phần mềm
  - Chỉ cho phép các VMs kết nối với nhau, nếu chúng kết nối vào cùng mạng (xác định qua tên)
- 🪜 **Host-only**: Trên Host mở một cổng ảo (tương tự loopback), kết nối vào một mạng ảo, ở đó, các VMs có thể cùng gia nhập
  - Kết nối này chỉ cho phép kết nối nội bộ (không ra mạng bên ngoài)
- Xem thêm về [Networking Modes trong VirtualBox](https://www.virtualbox.org/manual/ch06.html)

Nếu bạn không muốn hiểu về cách thức chúng hoạt động, bảng dưới đây thể hiện khả năng kết nối của từng mode

| Mode      | VM -> Host | VM <- Host   | VM1 <-> VM2 | VM -> Net/LAN | VM <- Net/LAN |
| --------- | ---------- | ------------ | ----------- | ------------- | ------------- |
| NAT       | **+**      | Port Forward | **-**       | **+**         | Port Forward  |
| Bridged   | **+**      | **+**        | **+**       | **+**         | **+**         |
| Internal  | **-**      | **-**        | **+**       | **-**         | **-**         |
| Host-only | **+**      | **+**        | **+**       | **-**         | **-**         |

> [!important]
> Theo bảng trên, mình cần kết nối ứng dụng RoR trong Virtual Machine và Host (để truy cập vào Postgres DB)
>
> - Có thể sử dụng Bridge, nhưng mode này quá rộng, cho phép cả kết nối từ LAN.
> - Do đó, chế độ hợp lý là **Host-only**

🔧 **Cấu hình cho Host-only Network**

- Ảnh dưới đây là cấu hình cho adapter trên Host cho mạng này.
  ![Config Host Adapter](assets/til/vm-host-only-adapter.png)
- Có thể thấy, địa chỉ IP của Host là **192.168.1.100**
- VM cũng cần được gắn một địa chỉ IP, ta có thể cấu hình thủ công, hoặc để một DHCP Server cấp phát tự động cho nó
  ![Config DHCP](assets/til/vm-host-only-dhcp.png)
- Ở đây, chạy một DHCP Server (tại địa chỉ _192.168.1.2_) cấp phát dải IP _192.168.1.3 -> 192.168.1.254_
- Kiểm tra địa chỉ IP trên VM tại cổng mạng tương ứng. VD: Cổng mạng mở trên VirtualBox là cổng thứ 3 thì cổng mạng trên VM có thể là _enp0s9_ - (thứ tự là 3,8,9,10)
- Hiện tại, IP được cấp cho VM của mình là **192.168.1.17**

✅ **Kiểm tra kết nối**

- Để kiểm tra kết nối, thông thường sử dụng hai lệnh

```cmd
# `ping` use ICMP Packet (Upper of IP on Layer 3 in TCP/IP)
ping <ip_addr>

# `telnet` (client-server service, less secure version of SSH)
telnet <ip_addr> <port>
```

- Thông thường, kết nối từ Host sang VM thông, song chiều ngược lại thì không.
- Lý do là vì Host (ở đây là Windows) đang có Firewall Rule chặn kết nối. Do đó, cần tạo một Rule mới (không nên tắt Firewall)

📏 **Thêm Rule cho Firewall/iptables**

Để kết nối từ VM tới cổng 5432 trên Host (đã mở để kết nối tới Postgres), thêm Rule sau cho Host

| Field          | Value                                   |
| -------------- | --------------------------------------- |
| Type           | **Inbound Rule**                        |
| Action         | **Allow**                               |
| Local Address  | Any                                     |
| Remote Address | Range: **192.168.1.3 -> 192.168.1.254** |
| Protocol       | TCP                                     |
| Local Port     | **5432**                                |
| Remote Port    | Any                                     |

#### 🚀 Run

Khác với môi trường thông thường, Postgres và RoR ở đây nằm trên hai máy khác nhau, ta cần chỉ định lại **connectionString** trong tệp **config/database.yml**

```yaml
# config/database.yml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

  username: <%= ENV["POSTGRES_USERNAME"] %>
  password: <%= ENV["POSTGRES_PASSWORD"] %>
  host: <%= ENV["POSTGRES_HOST"] %>
  port: <%= ENV["POSTGRES_PORT"] %>

development:
  <<: *default
  database: <db>_development

test:
  <<: *default
  database: <db>_test

production: # ...
```

Cụ thể:

```cmd
% Create from Docker environment
export POSTGRES_USERNAME=<username-to-connect>
export POSTGRES_PASSWORD=<password-to-connect>

export POSTGRES_HOST="192.168.1.100" % Host Address
export POSTGRES_PORT="5432" % Host Port
```

> [!note]
> Có thể sử dụng gem **`Figaro`**, khi đó, giá trị các biến này được lưu trong `application.yml` (gitignore)

📬 Cuối cùng, hãy chạy Postgres Container và chạy thử `rails db:create` để kiểm tra kết quả.

#### 🪝Tham khảo

- [DigitalOcean - Install RoR](https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-20-04)
- [Docker - Postgres](https://www.docker.com/blog/how-to-use-the-postgres-docker-official-image/)
- [VirtualBox - Virtual Networking](https://www.virtualbox.org/manual/ch06.html)
