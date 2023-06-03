# Cosmos 概览

## 一、选择题（多选）

1. Cosmos Network 是指什么？(ABCD)
A. Cosmos Hub 主网
B. 区块链的互联网
C. ATOM 区块链网络
D. 多个区块链网络组成的超级网络

2. 以下哪些不是跨链基金会的议程（Program）(B)
A. 促进发展负责任的监管环境
B. 提升 ATOM 代币的价格
C. 为全球 Builders 提供全方位支持
D. 提供 Cosmos 技术栈相关的在线培训

3. 以下哪些不是应用专有链的特性 (A)
A. 可升级性
B. 可扩展性
C. 高可用性
D. 独立自主性
E. 可互操作性

4. 以下哪些不是 Cosmos 技术栈的核心产品 (B)
A. Tendermint
B. Substrate
C. Cosmos SDK
D. IBC 跨链协议
E. Ignite CLI

5. 以下哪些说法是正确的 (AD)
A. Cosmos SDK 被全球超过 200 个 PoS 区块链项目采用
B. Tendermint 是支持拜占庭容错的 PoW 共识引擎
C. IBC 是基于哈希锁机制的跨链协议
D. Tendermint 被全球超过 40% 的 PoS 区块链网络采用

6. 以下哪些 ABCI 方法不是用于执行交易的 (CD)
A. BeginBlock
B. CheckTx
C. EndBlock
D. DeliverTx

7. 以下哪些是 Cosmos 链节点的功能分层 (ABDE)
A. 共识层
B. 连接层
C. 会话层
D. 网络层
E. 应用层

8. Tendermint Core 与应用层之间的通讯接口是 (B)
A. REST API
B. ABCI
C. Socket
D. GRPC
E. ABCI++

9. 以下哪些是 Tendermint Core 的特性 (BCE)
A. 交易的概率最终性
B. 拜占庭容错 BFT
C. 交易的快速最终性
D. 基于工作量证明 PoW
E. 交易的绝对最终性

10. 以下哪些说法是正确的 (BCD)
A. 通过向网络增加节点可提高网络的吞吐能力
B. ABCI++ 是对区块链网络的水平扩展
C. 应用专有链可对链上治理机制做定制化设计
D. IBC 是区块链之间的互操作协议

## 二、填空题

```text
1. Cosmos 的愿景是构建 ___ 区块链的互联网（The Internet of Blockchains）__
2. Cosmos Hub 的主网代号是 ____ Gaia _________, 主通证是 __ ATOM _______
3. Tendermint 是 _______ Jae Kwon Choe ____________ 发明的
4. Tendermint Core 正确运行需要至少占  __ 2/3 ____ 投票权的验证人不作恶
5. 如果你想授权他人代管你的链账户通证资产，你会使用 ___authz______ 模块功能
6. 如果你想替他人支付交易手续费，你会使用 ___feegrant______ 模块功能
7. 如果你想修改一个链上系统参数，你需要使用 ____gov_____ 模块功能
8. 如果你需要使用 Rust 语言编写智能合约，你会加载 _____cosmwasm_________ 合约模块
9. 如果你需要把以太坊上的通证跨链转移到 Cosmos 生态，你可以使用 ___Axelar_____ 或 ___Gravity Bridge_______ 网络
10. 如果你需要查看一笔跨链交易的分阶段详情，你可以使用 _____Mintscan_________ 或 ____IOBScan___________ 跨链浏览器
```

## 三、问答题

1. IBC 数据包为什么会超时？
网络延迟，带宽限制，区块链拥堵，数据包过大，配置不当。

2. 来自中国的技术团队为 IBC 协议贡献了哪些标准？
ICS-721, ICS-10, ICS-100, ICS-101

3. 跨链 NFT 转移协议（ICS-721）的 Go 语言实现代码库在哪儿？
<https://github.com/cosmos/cosmos-sdk/tree/main/x/nft>

4. 同时支持 Web, Android, iOS 平台的 Cosmos 生态钱包有哪些（不少于两个）？
Keplr, Cosmostation

5. Cosmos 中文技术社区翻译的关于 ABCI++ 的最近一篇微信公众号文章链接是什么？
<https://mp.weixin.qq.com/s/fC3d1vqcdpvGLeorNIpDUQ>
