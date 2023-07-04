# 跨链通信协议 IBC 应用

repo: <https://github.com/Akagi201/planet>

## ignite scaffold

```sh
ignite scaffold chain github.com/Akagi201/planet --no-module --address-prefix planet
cd planet
# 生成 blog 模块, 并集成 IBC
ignite scaffold module blog --ibc -y
# 给 blog 模块添加针对 post 的 CRUD
ignite scaffold list post title content creator --no-message --module blog -y
# 给 blog 模块添加针对 sentPost 的 CRUD
ignite scaffold list sentPost postID title chain creator --no-message --module blog -y
# 给 blog 模块添加针对 timedoutPost 的 CRUD
ignite scaffold list timedoutPost title chain creator --no-message --module blog -y
# 添加 IBC 发送数据包和确认数据包的结构
ignite scaffold packet ibcPost title content --ack postID --module blog -y
```

## manual modify code

修改 `proto/planet/blog/packet.proto` 中 `IbcPostPacketData`, 添加 `Creator`, `ignite chain build` 生成 pb.go

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
  - 10000token
  - 1000000000stake
- name: bob
  coins:
  - 5000token
  - 1000000000stake
faucet:
  name: bob
  coins:
  - 500token
  - 10000000stake
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
  - 10000token
  - 10000000000stake
- name: bob
  coins:
  - 5000token
  - 1000000000stake
faucet:
  name: bob
  coins:
  - 500token
  - 10000000stake
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
planetd tx blog send-ibc-post blog channel-0 "Hello" "Hello Mars, I'm Alice from Earth" --from alice --chain-id earth --home ~/.earth
```

## 通过 rpc 查询结果

```sh
planetd q blog list-post --node tcp://localhost:26659

planetd q blog list-sent-post
```

## add updatePost

```sh
ignite scaffold packet ibcUpdatePost title content --ack ok:bool --module blog -y
```

## 不使用 ignite

初始化链

```sh
planetd init earth --chain-id earth --home ~/.earth
```

创建 alice 地址

```sh
planetd keys add alice --keyring-backend test --home ~/.earth
```

```sh
- address: planet16a2jq2epzratvrs606yxfrncsugd4u77j7hwju
  name: alice
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"Ak/w8LCV8c1WJPGxkoky17LVaMqhvoao0gTCnkZ2X2me"}'
  type: local


**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

file gasp cabbage bench flat truly nice turkey manual chase view meat shiver leaf rather mansion accuse steel always battle toilet clerk hole town
```

为 alice 在 genesis 中分配 token

```sh
planetd add-genesis-account alice 20000token,200000000stake --home ~/.earth --keyring-backend test
```

通过质押让 alice 成为 validator

* 将 alice 注册为 validator operator account
* 为自己质押相应的 token(self-delegates)
* 将 operator account 与本地节点的 pub key 做关联。

```sh
planetd gentx alice 100000000stake --chain-id earth --home ~/.earth --keyring-backend test
```

将生成的交易写入到 genesis 文件中

```sh
planetd collect-gentxs --home ~/.earth
```

启动节点

```sh
planetd start --home ~/.earth
```

类似地，启动 mars 链

```sh
planetd init mars --chain-id mars --home ~/.mars
planetd keys add alice --keyring-backend test --home ~/.mars
planetd add-genesis-account alice 20000token,200000000stake --home ~/.mars --keyring-backend test
planetd gentx alice 100000000stake --chain-id mars --home ~/.mars --keyring-backend test
planetd collect-gentxs --home ~/.mars
planetd start --home ~/.mars
```

```sh
- address: planet1e35a3jp8tyqew0plmxzdej8kjxfegcwrzk0knc
  name: alice
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"A+9TYEG2gAEmWhuC3gpV9X3/tY1zqCdXSPaP+r2DOHtW"}'
  type: local


**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

talent wheat suspect tunnel submit sword trouble private stone mammal slam sight buddy embark thought rubber control seek measure sound opinion item toddler engage
```

修改 earth 与 mars 的配置文件的端口，避免冲突 (不明原因，上面初始化链时候配置的端口没生效，可能只对 ignite 有效？)

```sh
~/.earth/config/config.toml
~/.earth/config/app.toml
~/.mars/config/config.toml
~/.mars/config/app.toml
```

初始化 relayer 配置

```sh
rly config init --home ~/.relayer
```

修改 `~/.relayer/config/config.yaml`，添加两条链的配置

```yaml
global:
    api-listen-addr: :5183
    timeout: 10s
    memo: "planet"
    light-cache-size: 20
chains:
    earth:
        type: cosmos
        value:
            key: alice
            keyring-backend: test
            chain-id: earth
            rpc-addr: http://localhost:26661
            account-prefix: planet
            gas-adjustment: 1.3
            gas-prices: 0.01token
            min-gas-amount: 0
            debug: false
            timeout: 20s
            block-timeout: ""
            output-format: json
            sign-mode: direct
            extra-codecs: []
            coin-type: 118
            broadcast-mode: batch
    mars:
        type: cosmos
        value:
            key: alice
            keyring-backend: test
            chain-id: mars
            rpc-addr: http://localhost:26659
            account-prefix: planet
            gas-adjustment: 1.3
            gas-prices: 0.01token
            min-gas-amount: 0
            debug: false
            timeout: 20s
            block-timeout: ""
            output-format: json
            sign-mode: direct
            extra-codecs: []
            coin-type: 118
            broadcast-mode: batch
paths:
    planet:
        src:
            chain-id: earth
        dst:
            chain-id: mars
        src-channel-filter:
            rule: ""
            channel-list: []
```

导入私钥

```sh
rly keys restore earth alice 'file gasp cabbage bench flat truly nice turkey manual chase view meat shiver leaf rather mansion accuse steel always battle toilet clerk hole town'
rly keys restore mars alice 'talent wheat suspect tunnel submit sword trouble private stone mammal slam sight buddy embark thought rubber control seek measure sound opinion item toddler engage'
```

查看 key

```sh
rly keys list earth
rly keys list mars
```

创建轻客户端

```sh
rly tx clients planet
```

创建 connection

```sh
rly tx connection planet
```

创建 channel

```sh
rly tx channel planet --src-port blog --dst-port blog --order unordered
```

启动 relayer

```sh
rly start --home ~/.relayer
```
