nakama服务端部署

```java
基础部分
1 迁移/绑定数据库 for PosgreSQL
./nakama migrate up --database.address ysr:123456@127.0.0.1:5432
2.1 启动服务器 for PostgreSQL
./nakama --database.address ysr:123456@127.0.0.1:5432
2.2 通过配置文件启动服务器
sudo ./nakama --config ./data/path/to/nakama-config.yml
（nakama --config xxxx.yml） 启动时设置配置文件

// 游戏管理端控制台 默认账号密码 admin:password
*也可以通过systemd成为系统服务

服务器配置
// 相关端口 默认
7530 HTTP API
7351 游戏控制台
7349 gRPC API
7348 gRPC API server for the embedded （嵌入式？）

相关配置参考 https://heroiclabs.com/docs/nakama/getting-started/configuration/

nakama相关指令
migrate // 迁移
migrate up  创建数据库架构并将其更新为 Nakama 所需的最新版本。默认情况下，模式会按顺序更新到最新的可用模式
migrate status 提供有关当前应用于数据库的模式的信息，以及是否有未应用的模式。
migrate down 将数据库模式降级到请求的版本。默认情况下，它一次降级一个架构更改。
migrate redo 降级一项架构更改，然后重新应用更改。

database.address 要连接的数据库节点。它应该遵循username:password@address:port/dbname（postgres://协议自动附加到路径）的形式。默认为root@localhost:26257.
--limit 无论是在运行时使用的迁移数up，down或redo。

config  // 替换配置
nakama --config path/to/xxx.yml --database.address root@localhost:26257 --database.address root@machine-2:26257

具体指令参考 https://heroiclabs.com/docs/nakama/getting-started/configuration/
```

```yaml
#配制文件相关

name: nakama-node-1              # 节点名称（必须是唯一的）
data_dir: "./data/"              # Nakama 将存储其数据（包括日志）的可写文件夹的绝对路径。默认值是启动 Nakama 的工作目录。

logger:
  stdout: false                     # 将日志重定向到控制台标准输出。将不再使用日志文件。默认为true。
  level: "warn"                     # 要生成的最低日志级别。值是debug，info，warn和error。默认为info。
  file: "./data/path/to/logfile.log"  # 将输出记录到文件（以及stdout是否已设置）。确保目录和文件是可写的。

metrics: # 指标？
  reporting_freq_sec: 60            # 指标导出的频率。默认值为 60 秒。
  namespace: ""                     # Prometheus 的命名空间或 Stackdriver 指标的前缀。它总是在节点名称前面。默认值为空。
  prometheus_port: 0                # 暴露 Prometheus 的端口。默认值为“0”，禁用 Prometheus 导出。

database:
  address:
  - "ysr:123456@localhost:5432"          # 要连接的数据库节点列表。它应该遵循username:password@address:port/dbname（postgres://协议自动附加到路径）的形式。默认为root@localhost:26257.
  conn_max_lifetime_ms: 0           # 在连接被终止并创建新连接之前重用数据库连接的时间（以毫秒为单位）。默认值为 0（不确定）。
  max_open_conns: 0                 # 允许打开的数据库连接的最大数量。默认值为 0（无限制）。
  max_idle_conns: 100               # 允许打开但未使用的数据库连接的最大数量。默认值为 100。

runtime:
  env:                                        # 作为环境变量公开给运行时脚本的键值属性列表。
  - "example_apikey=example_apivalue"
  - "encryptionkey=afefa==e332*u13=971mldq"
  path: "/tmp/modules/folders"                # 服务器在启动时扫描和加载的模块路径。默认值为data_dir/modules。
  http_key: "testhttpkey"                  # 用于验证 HTTP 运行时调用的密钥。默认值为defaultkey。

socket:
  server_key: "testsocketkey"                 # 用于建立与服务器的连接的服务器密钥。默认值为defaultkey
  port: 7350
  max_message_size_bytes: 4096 # 每条消息允许从客户端套接字读取的最大数据量（以字节为单位）。用于实时连接。默认值为 4096。     
  read_timeout_ms: 10000       # 读取整个请求的最大持续时间（以毫秒为单位）。用于 HTTP 连接。默认值为 10000。
  write_timeout_ms: 10000      # 超时写入响应之前的最大持续时间（以毫秒为单位）。用于 HTTP 连接。默认值为 10000。
  idle_timeout_ms: 60000       # 启用保持活动时等待下一个请求的最长时间（以毫秒为单位）。用于 HTTP 连接。默认值为 60000。
  write_wait_ms: 5000          # 写入数据时等待来自客户端的确认的时间（以毫秒为单位）。用于实时连接。默认值为 5000。
  pong_wait_ms: 10000          # 发送 ping 后等待来自客户端的 pong 消息的时间（以毫秒为单位）。用于实时连接。默认值为 25000。
  ping_period_ms: 8000         # Must be less than pong_wait_ms 客户端 ping 消息之间的等待时间（以毫秒为单位）。该值必须小于pong_wait_ms。用于实时连接。默认值为 15000。
  outgoing_queue_size: 16      # 等待发送到客户端的最大消息数。如果超过这个值，客户端会被认为太慢并且会断开连接。在处理实时连接时使用。默认值为 16。

social:  # 社交网络获取用户信息
  steam:  # steam
  publisher_key: ""
  app_id: 0

console:     # 嵌入式开发者控制台相关的配置
  port: 7351
  username: "admin"
  password: "password"

cluster:   # 节点应如何相互连接以形成集群
  join:
  - "10.0.0.2:7352"
  - "10.0.0.3:7352"
  gossip_bindaddr: "0.0.0.0"
  gossip_bindport: 7352
  rpc_port: 7353
  local_priority: true
  work_factor_interval_ms: 1000

matchmaker: # 赛事匹配/房间匹配
  max_tickets: 2      # 每个会话或聚会允许的最大并发匹配票数。默认 3
  interval_sec: 15    # 媒人尝试形成比赛的速度，以秒为单位。默认 15
  max_intervals: 3    # 在允许最小计数之前，匹配器尝试在最大玩家计数处查找匹配的时间间隔。默认 2。

iap:    # 应用内购买
  apple:
    shared_password: "password"
  google:
    client_email: "email@google.com"
    private_key: "pk"
  huawei:
    public_key: "pk"
    client_id: "id"
    client_secret: "secret"
```

### Full Source Builds

The codebase uses Protocol Buffers, GRPC, GRPC-Gateway, and the OpenAPI spec as part of the project. These dependencies are generated as sources and committed to the repository to simplify builds for contributors.

To build the codebase and generate all sources follow these steps.

1. Install the toolchain.

   ```shell
   go install \
       "google.golang.org/protobuf/cmd/protoc-gen-go" \
       "google.golang.org/grpc/cmd/protoc-gen-go-grpc" \
       "github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway" \
       "github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2"
   ```

2. If you've made changes to the embedded Console.

    ```shell
    cd console/ui
    ng serve
    ```

3. Re-generate the protocol buffers, gateway code and console UI.

   ```shell
   env PATH="$HOME/go/bin:$PATH" go generate -x ./...
   ```

4. Build the codebase.

   ```shell
   go build -trimpath -mod=vendor
   ```



nakama服务相关接口





PostgreSQL 数据库相关指令

```
which psql
pg_ctl -V
ps -ef |grep postgres
pg_ctl start
\du
```






客户端基础接口

1. 链接服务端
   var client = new Nakama.Client("http", "127.0.0.1", 7350, "defaultKey");

2. socket链接
   var socket = client.NewSocket(); 
   bool appearOnline = true; int connectionTimeout = 30; 
   await socket.ConnectAsync(Session, appearOnline, connectionTimeout);

3. 身份验证

   // 几种身份验证 

   1）通过设备ID登陆

   2）FaceBook身份登陆
   3）可以储存一部分客户信息进入session
   4）session的生命周期的保持 （会话状态的确认判断是否过期）

   5）注销会话

4. 账户相关
