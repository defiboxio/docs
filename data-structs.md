# Defibox数据结构说明

这篇文章将介绍Defibox Swap的数据结构，方便第三方开发者接入。

### 合约名

Defibox Swap的合约名是：swap.defi

可以借助bloks直接查看合约的abi信息：[点击查看](https://bloks.io/account/swap.defi?loadContract=true&tab=Tables&account=swap.defi&scope=swap.defi&limit=100)

### 基础类型

#### Token

| 字段     | 类型   | 说明               |
| -------- | ------ | ------------------ |
| contract | name   | 合约名             |
| symbol   | symbol | 币种，带符号和精度 |



### 流动池交易对 Pair

#### 说明

pair表是用于存放所有交易对数据的表，是整个合约的核心部分。

#### 表结构

Table: pairs

Scope: swap.defi

| 字段            | 类型       | 说明                     |
| --------------- | ---------- | ------------------------ |
| id              | uint64_t   | 交易对ID                 |
| token0          | Token      | 币种1信息                |
| token1          | Token      | 币种2信息                |
| reserve0        | asset      | 币种1的资产总量          |
| reserve1        | asset      | 币种2的资产总量          |
| liquidity_token | uint64_t   | 交易对做市凭证总数       |
| price0_last     | double     | 最近一笔交易价格         |
| price1_last     | double     | 最近一笔交易价格（反向） |
| block_time_last | time_point | 上次更新的时间           |

#### 请求示例：

```shell
curl -X POST -d '{"code":"swap.defi","scope":"swap.defi","table":"pairs","json":true,"limit":10}' -H 'Content-Type: application/json' https://eos.newdex.one/v1/chain/get_table_rows
```



### 配置表 Config

#### 说明

config表可以存储一些配置，比如手续费比例等。

#### 表结构

Table: config

Scope: swap.defi

| 字段         | 类型     | 说明                        |
| ------------ | -------- | --------------------------- |
| status       | uint8_t  | 状态，0为正常，其它为暂     |
| pair_id      | uint64_t | pairs表的最大ID，用于ID自增 |
| trade_fee    | uint8_t  | 交易手续费，计算时除以10000 |
| protocol_fee | uint8_t  | 协议手续费，计算时除以10000 |
| fee_account  | name     | 协议费手续费账号            |

#### 请求示例：

```shell
curl -X POST -d '{"code":"swap.defi","scope":"swap.defi","table":"config","json":true,"limit":1}' -H 'Content-Type: application/json' https://eos.newdex.one/v1/chain/get_table_rows
```

#### 

### 订单表 Order

#### 说明

order表是用于存储临时的订单，用户做市需要充值两个资产，用表用于临时记录用户的资产。调用deposit方法后，order表的数据将会清除。

#### 表结构

Table: orders

Scope: pari_id

| 字段      | 类型  | 说明                        |
| --------- | ----- | --------------------------- |
| owner     | name  | 状态，0为正常，其它为暂     |
| quantity0 | asset | pairs表的最大ID，用于ID自增 |
| quantity1 | asset | 交易手续费，计算时除以10000 |

#### 请求示例：

```shell
curl -X POST -d '{"code":"swap.defi","scope":1,"table":"orders","json":true,"limit":10}' -H 'Content-Type: application/json' https://eos.newdex.one/v1/chain/get_table_rows
```

### 

还有一个余额表，用于核对合约的各币种余额，这里不再介绍。