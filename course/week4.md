# 跨链通信协议 IBC 应用

## ignite scaffold

```sh
ignite scaffold chain planet --no-module --address-prefix planet
cd planet
# 生成 blog 模块, 并集成 IBC
ignite scaffold module blog --ibc
# 给 blog 模块添加针对 post 的 CRUD
ignite scaffold list post title content creator --no-message --module blog
# 给 blog 模块添加针对 sentPost 的 CRUD
ignite scaffold list sentPost postID title chain creator --no-message --module blog
# 给 blog 模块添加针对 timedoutPost 的 CRUD
ignite scaffold list timedoutPost title chain creator --no-message --module blog
# 添加 IBC 发送数据包和确认数据包的结构
ignite scaffold packet ibcPost title content --ack postID --module blog
```

## manual modify code

修改 `proto/blog/packet.proto` 中 `IbcPostPacketData`, 添加 `Creator`, `ignite chain build` 生成 pb.go

在 `x/blog/keeper/msg_server_ibc_post.go` 中发送数据包前更新 `Creator`.

keeper 中 `OnRecvIbcPostPacket`

```go
id := k.AppendPost(
    ctx,
    types.Post{
        Creator: packet.SourcePort + "-" + packet.SourceChannel + "-" + data.Creator,
        Title:   data.Title,
        Content: data.Content,
    },
)

packetAck.PostID = strconv.FormatUint(id, 10)
```

keeper 中 `OnAcknowledgementIbcPostPacket`

```go
k.AppendSentPost(
    ctx,
    types.SentPost{
        Creator: data.Creator,
        PostID:  packetAck.PostID,
        Title:   data.Title,
        Chain:   packet.DestinationPort + "-" + packet.DestinationChannel,
    },
)
```

keeper 中 `OnTimeoutIbcPostPacket`

```go
k.AppendTimedoutPost(
    ctx,
    types.TimedoutPost{
        Creator: data.Creator,
        Title:   data.Title,
        Chain:   packet.DestinationPort + "-" + packet.DestinationChannel,
    },
)
```

## 添加链启动配置文件

earth.yml

```yaml
version: 1
build:
  proto:
    path: proto
    third_party_paths:
    - third_party/proto
    - proto_vendor
accounts:
- name: alice
  coins:
  - 1000token
  - 100000000stake
- name: bob
  coins:
  - 500token
  - 100000000stake
faucet:
  name: bob
  coins:
  - 5token
  - 100000stake
  host: 0.0.0.0:4500
genesis:
  chain_id: earth
validators:
- name: alice
  bonded: 100000000stake
  home: $HOME/.earth
```

mars.yml

```yaml
version: 1
build:
  proto:
    path: proto
    third_party_paths:
    - third_party/proto
    - proto_vendor
accounts:
- name: alice
  coins:
  - 1000token
  - 1000000000stake
- name: bob
  coins:
  - 500token
  - 100000000stake
faucet:
  name: bob
  coins:
  - 5token
  - 100000stake
  host: :4501
genesis:
  chain_id: mars
validators:
- name: alice
  bonded: 100000000stake
  app:
    api:
      address: :1318
    grpc:
      address: :9092
    grpc-web:
      address: :9093
  config:
    p2p:
      laddr: :26658
    rpc:
      laddr: :26659
      pprof_laddr: :6061
  home: $HOME/.mars

```

## 启动两条链

one terminal

```sh
ignite chain serve -c earth.yml
```

another terminal

```sh
ignite chain serve -c mars.yml
```

## 生成 relayer 配置

```sh
rm -rf ~/.ignite/relayer
ignite relayer configure -a \
  --source-rpc "http://0.0.0.0:26657" \
  --source-faucet "http://0.0.0.0:4500" \
  --source-port "blog" \
  --source-version "blog-1" \
  --source-gasprice "0.0000025stake" \
  --source-prefix "planet" \
  --source-gaslimit 300000 \
  --target-rpc "http://0.0.0.0:26659" \
  --target-faucet "http://0.0.0.0:4501" \
  --target-port "blog" \
  --target-version "blog-1" \
  --target-gasprice "0.0000025stake" \
  --target-prefix "planet" \
  --target-gaslimit 300000
```

## 启动 relayer

```sh
ignite relayer connect
```

## 从 earth 链向 mars 链发送 blog 数据包

```sh
planetd tx blog send-ibc-post blog channel-4 "Hello" "Hello Mars, I'm Alice from Earth" --from alice --chain-id earth --home ~/.earth
```

## 通过 rpc 查询结果

```sh
planetd q blog list-post --node tcp://localhost:26659

planetd q blog list-sent-post
```
