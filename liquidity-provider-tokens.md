# Defibox LP Tokens设计文档



## 简介

Defibox采用类似Uniswap的设计，用户在Defibox做市后将得到真实可流通的LP（Liquidity Provider），本文档将介绍开发者如何接入 Defibox LP Tokens.

下面将介绍接入LP的两种方式。

### 做市凭证

做市凭证(LP Token)是代表用户在流动池的做市份额，也是取回做市资产的一种证明。
当用户参与做市，将立刻获得对应的做市凭证，它是标准的EOS代币，可自由转移，不同的做市凭证对应不同流动池。
拥有做市凭证的EOS账户可自由取回对应的做市资产，同时还可领取对应的做市挖矿奖励( BOX )。
当用户将做市凭证转移给其它EOS用户时，对应做市资产的取回权以及后续做市挖矿奖励的领取权也将随之转移。
第三方DeFi协议会推出基于Defibox做市凭证的应用场景，帮助用户盘活做市资产或者获得多重收益。
Defibox无法对任何第三方DeFi协议进行安全性判断，也无法阻止第三方DeFi协议推出基于Defibox做市凭证的应用场景，因此，强烈建议用户在仔细辨别第三方DeFi协议的安全性后，谨慎决定是否转移做市凭证。

### 映射凭证

Defibox除了推出做市凭证外，同时还支持第三方DeFi协议采用映射Defibox做市凭证的方法推出应用场景。
Defibox智能合约中有一张数据表专门用于记录每个流动池的做市商信息，包括：做市商EOS账户名、做市数量等，相关信息在链上均为公开可查询的状态，任何第三方DeFi协议都可以自由获取。
采用映射Defibox做市凭证的方法，第三方DeFi协议在推出应用场景的同时不需要额外考虑做市挖矿奖励（BOX）的再分配问题，也可免去管理做市凭证的安全性问题，用户的参与也更简单。



## 接入

### 相关合约

Defibox Swap合约：swap.defi

LP Token合约：lptoken.defi

BOX Token合约：token.defi

### LP命名规则

所有流动池的LP Token都是独立的，在Uniswap因为每个流动池都是单独合约，因此他们可以直接叫UNI或者UNI-v2，在我们的设计中，所有的LP Tokens都在同一个合约下发布的，因此币名各不相同。

Defibox LP Token都是以BOX开头，然后加上流动池的pair_id转成26进制（字母），例如流动池ID是1，转换后就是BOXA，如果是27，对应的是BOXAA，以此类推。最大的币名为BOXZZZZ，可以容纳将近50万个流动池。

### 接入映射凭证(Liquidity Provider Mapping)

映射凭证接入方法很简单，开发者只需要读取合约内的做市记录即可，这种方式无法锁定用户的LP，开发者需要定时读取加权平均等手段去计算用户的做市数量。具体信息如下：

合约名：lptoken.defi

表名：userinfo

Scope: 币名，例如：BOXL

获取某个流动池的做市列表

```shell
curl -X POST -d '{"code":"lptoken.defi","scope":"BOXL","table":"userinfo","json":true,"limit":-1}' -H 'Content-Type: application/json' https://eos.newdex.one/v1/chain/get_table_rows
```

获得数组信息如下：

| 字段      | 类型     | 说明                             |
| --------- | -------- | -------------------------------- |
| owner     | name     | 用户名                           |
| liquidity | uint64_t | 做市凭证数量                     |
| debt      | uint64_t | 用于记录已领奖励的数值（可忽略） |

### 接入直实做市凭证(Liquidity Provider)

接入真实做市凭证需要开发者接收并保管LP，此LP是取回用户在Defibox做市资金的唯一凭证，相当重要。第三方应用如抵押类应用可以代替资产直接使用。

#### BOX挖矿奖励的分配

在Defibox做市可享受做市挖矿的BOX奖励，但BOX奖励只分配给LP持有者，如果LP发生转移，系统会马上结算，之后的奖励将归属于新的LP持有者。接入做市凭证的难点在于如何二次分配这些奖励。

奖励在用户没有操作的时候不会分配，只有用户余额发生变化（转入或者转出都可以）或者用户主动去更新奖励才会进行分配。分配后的数量会在reward表中，具体信息如下：

合约名：lptoken.defi

表名：rewards

Scope: lptoken.defi

| 字段       | 类型     | 说明         |
| ---------- | -------- | ------------ |
| owner      | name     | 用户名       |
| cumulative | uint64_t | 历史累计奖励 |
| unclaimed  | uint64_t | 尚未领取奖励 |

#### 主动更新奖励
参数：
| 字段       | 类型     | 说明         |
| ---------- | -------- | ------------ |
| code      | symbol_code     | 币名，例如 BOXL       |
| owner | name | 要领取奖励的用户 |

示例：
```shell
cleos -u https://eos.newdex.one push action lptoken.defi update '["BOXL","eostestuser1"]' -p eostestuser1
```

#### 领取奖励
参数：
| 字段       | 类型     | 说明         |
| ---------- | -------- | ------------ |
| owner | name | 要领取奖励的用户 |

示例：
```shell
cleos -u https://eos.newdex.one push action lptoken.defi claim '["eostestuser1"]' -p eostestuser1
```

#### 更新所以流动池并领取奖励
参数：
| 字段       | 类型     | 说明         |
| ---------- | -------- | ------------ |
| owner | name | 要领取奖励的用户 |
| offset | symbol_code | 从哪个币开始领取，不填可留空 |
| limit | uint16_t | 更新池子的最大数量 |

示例：
```shell
cleos -u https://eos.newdex.one push action lptoken.defi claimall '["eostestuser1","BOXA",10]' -p eostestuser1
```

特别说明：因为EOS网络的特性，执行过于复杂的运算会导致交易失败，因此offset配合limit可以分批次执行。
