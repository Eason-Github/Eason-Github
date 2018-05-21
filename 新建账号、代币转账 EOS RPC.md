### **前言**

 最近在研究 EOS 的 RPC API，但是由于官方API文档的不够详尽，新建账号（new account）这一个操作就折腾了一个多星期。皇天不负有心人，终于调通了新建账号，代币转账也轻松解决。特地写这篇文章（适用于 EOS dawn 4.0 和 4.1），帮助准备使用 EOS RPC 做 Dapp 开发的朋友，如有问题，欢迎批评指正。

感谢欧链朋友的帮助，推荐他们的

 EOSDevlepor（EOS开发助手） https://github.com/OracleChain/EOSDevHelper

 PocketEOS （EOS 钱包）	        https://pocketeos.com/#/website-pc

和 另外一款 安卓调试工具  https://github.com/plactal/EosCommander

一个博客 http://www.cnblogs.com/Evsward 

（我的以太坊地址 0x0D11cb36d8F881e3c747B55B6B0452e7E89b3D9f ，欢迎打赏各种 token 😄） 



### **概述**

**代币转账** 和 **新建账号**的 sign_transaction、push_transaction 类似，主要就是 智能合约的不同 和 调用的action 的不同 以及  action 中具体的参数不同

新建账号示例  https://gist.github.com/Eason-Github/283fcaba2c2c100eeb2743900d694c0a

代币转账示例  https://gist.github.com/Eason-Github/c38aea19e889cc78390b9c01ff0a96bb



这篇文章主要梳理一下**「新建账号」**的流程和相关接口，代币转账具体参数可以看上面👆的链接

新建账号（newaccount）需要用 「已有的账号」 **创建**「 新账号」

内部是 **已有账号** 调用**系统智能合约eosio**中的 **newaccount** 的 action

新建账号的交易需要用 **创建者** 的 **私钥签名交易**（sign_transaction），然后 **推送签名后的交易**  (push_transaction)到区块链中



**a.  sign_transaction 图示**

![sign_transaction ](/Users/liu/Desktop/区块链100问/sign_transaction .png)

**b. push_transaction 图示**

![push_transaction](/Users/liu/Desktop/区块链100问/push_transaction.png)

### **具体接口**

**1、POST    http://127.0.0.1:8888/v1/chain/abi_json_to_bin （序列化新建账号的 json）**

**请求参数**：

| 参数名称 | 参数类型 |                   描述                   |
| :------: | :------: | :--------------------------------------: |
|   code   |  string  |      系统智能合约，默认填写“eosio”       |
|  action  |  string  | 智能合约中的action，默认填写“newaccount” |
| creator  |  string  |                  创建者                  |
|   name   |  string  |                新建账号名                |
|   key    |  string  |              新建账号的公钥              |

**请求示例**：

```json
{
  "code": "eosio",
  "action": "newaccount",
  "args": {
    "creator": "bitcoin",
    "name": "eason",
    "owner": {
      "threshold": 1,
      "keys": [
        {
          "key": "EOS4ufZoTw95yHJS6Cyz3h4w5a2W4cyYpMYRnd7gbFZuCfPxUFS6r", //owner public key
          "weight": 1
        }
      ],
      "accounts": [],   
      "waits": []      
    },
    "active": {
      "threshold": 1,
      "keys": [
        {
          "key": "EOS4ufZoTw95yHJS6Cyz3h4w5a2W4cyYpMYRnd7gbFZuCfPxUFS6r", //active public key
          "weight": 1
        }
      ],
      "accounts": [],    
      "waits": []        
    }
  }
}
```





**响应参数**

| 参数名称 | 参数类型 |                             描述                             |
| :------: | :------: | :----------------------------------------------------------: |
| binargs  |  string  | 序列化的结果，在sign_transaction 和 push_transaction 中作为 **data** 请求参数 |

**响应示例**

```json
{
    "binargs": "000000603a8ab23b000000ca3d364dfb0100000001000202aba1b7d9fc5de9dd93308dc5ebedcb066c8e5b36970bfd82ae715d9e8c3060010000000100000001000202aba1b7d9fc5de9dd93308dc5ebedcb066c8e5b36970bfd82ae715d9e8c306001000000"
}
```





**2、GET   http://127.0.0.1:8888/v1/chain/get_info （获取 EOS 区块链的最新区块号）**

**响应参数**

|    参数名称    | 参数类型 |    描述    |
| :------------: | :------: | :--------: |
| head_block_num |  number  | 最新区块号 |

**响应示例**

```json
{
    "server_version": "13952d45",
    "head_block_num": 359934,
    "last_irreversible_block_num": 359934,
    "last_irreversible_block_id": "a69af2c4aa56b5c4bd1cdf9c2acb1a7796bbc3043954e36da182a144ddcf58fb",
    "head_block_id": "a69af2c4aa56b5c4bd1cdf9c2acb1a7796bbc3043954e36da182a144ddcf58fb",
    "head_block_time": "2018-05-17T09:02:12",
    "head_block_producer": "eosio",
    "virtual_block_cpu_limit": 100000000,
    "virtual_block_net_limit": 1048576000,
    "block_cpu_limit": 99900,
    "block_net_limit": 1048576
}
```



**3、POST http://127.0.0.1:8888/v1/chain/get_block （获取最新区块的具体信息）**

**请求参数**

|    参数名称     | 参数类型 |                     描述                      |
| :-------------: | :------: | :-------------------------------------------: |
| block_num_or_id |  number  | 最新区块号，上一个响应结果中的 head_block_num |

```json
{
  "block_num_or_id":359934
}
```

**响应参数**

|     参数名称     | 参数类型 |                             描述                             |
| :--------------: | :------: | :----------------------------------------------------------: |
|    timestamp     |  string  |                      最新区块的生成时间                      |
|    block_num     |  number  | 区块号，作为sign_transaction 和 push_transaction中的 **ref_block_num**请求参数 |
| ref_block_prefix |  number  | 作为sign_transaction 和 push_transaction中的  **ref_block_prefix** 请求参数 |

**响应示例**

```json
{
    "timestamp": "2018-05-17T09:02:12.500",
    "producer": "eosio",
    "confirmed": 0,
    "previous": "00057dfd5044aba0d750eff1fbb84ac92cbf29db1354968816fd2a9aefb0a0b4",
    "transaction_mroot": "0000000000000000000000000000000000000000000000000000000000000000",
    "action_mroot": "dee87e5d025383574ac12c310faf6b759fba52bd19977399b7ebf6ccdd81c7fa",
    "schedule_version": 0,
    "header_extensions": [],
    "producer_signature": "SIG_K1_KVX3RRTS4ch9m6bWDctsAhDWtFydTrg3mW7PaqCXnBZZWezBW23enggeW4ijuWBHBVsDoxzjMvspoFtPsU5nmau4ZYomZo",
    "transactions": [],
    "block_extensions": [],
    "id": "a69af2c4aa56b5c4bd1cdf9c2acb1a7796bbc3043954e36da182a144ddcf58fb",
    "block_num": 359934,
    "ref_block_prefix": 1943477914
}
```



**4、POST   http://127.0.0.1:8888/v1/wallet/unlock （解锁钱包，签名交易前，需要解锁账号所在的钱包）**

**请求参数**

| 参数名称 | 参数类型 |   描述   |
| :------: | :------: | :------: |
|          |  string  | 钱包名称 |
|          |  string  | 钱包密码 |

**请求示例**

```json
["liu","PW5KjWHnhL5kSRxpWyHQj321dFsZN62HAbZjVSqnDvzKMuEKBZ1T9"]
```

**响应示例** 

```
{}  //成功解锁钱包，返回{}
```





**5、POST  http://127.0.0.1:8888/v1/wallet/sign_transaction（签名新建账号的交易）**

**请求的参数**

|     参数名称     | 参数类型 |                      描述                      |
| :--------------: | :------: | :--------------------------------------------: |
|  ref_block_num   |  number  |              上面获得的最新区块号              |
| ref_block_prefix |  number  |          上面获得的最新区块号相关信息          |
|    expiration    |  string  | 过期时间 = timestamp 加上 一段时间 ，例如1分钟 |
|     account      |  string  |      调用系统智能合约账号名，默认为 eosio      |
|       name       |  string  |      新建账号的action，默认为 newaccount       |
|      actor       |  string  |                 创建者 账户名                  |
|       data       |  string  |     abi_json_to_bin 序列化后的 值 binargs      |
|                  |  string  |                 创建者的 公钥                  |

**请求示例**

```json
[
  {
    "ref_block_num": 363759,
    "ref_block_prefix": 4033496171,
    "expiration": "2018-05-17T09:54:06.500",
    "actions": [
      {
        "account": "eosio",  //有 newaccount 的 action 的智能合约账号
        "name": "newaccount",
        "authorization": [
          {
            "actor": "bitcoin",
            "permission": "active"
          }
        ],
        "data": "000000603a8ab23b000000ca3d364dfb0100000001000202aba1b7d9fc5de9dd93308dc5ebedcb066c8e5b36970bfd82ae715d9e8c3060010000000100000001000202aba1b7d9fc5de9dd93308dc5ebedcb066c8e5b36970bfd82ae715d9e8c306001000000" // //abi_json_to_bin 的响应参数 binargs
      }
    ],
    "signatures": []
  },
  [
    "EOS5wQ4HaFFDxyfc23dZNXUTGBHepM1vXGfr1vkfWHfRfvAMXP7VV" //创建者的公钥（交易发起者的公钥），其实是用的公钥对应的私钥进行签名的，签名前需要先解锁包含此私钥的钱包
  ],
  ""
]
```

**响应参数**

|  参数名称  | 参数类型 |                          描述                           |
| :--------: | :------: | :-----------------------------------------------------: |
| signatures |  string  | 新建账号的交易 的签名结果，最后 push_transaction 中使用 |

**响应示例**

```json
{
    "expiration": "2018-05-17T09:54:06",
    "ref_block_num": 36079,
    "ref_block_prefix": 4033496171,
    "max_net_usage_words": 0,
    "max_cpu_usage_ms": 0,
    "delay_sec": 0,
    "context_free_actions": [],
    "actions": [
        {
            "account": "eosio",
            "name": "newaccount",
            "authorization": [
                {
                    "actor": "bitcoin",
                    "permission": "active"
                }
            ],
            "data": "000000603a8ab23b000000ca3d364dfb0100000001000202aba1b7d9fc5de9dd93308dc5ebedcb066c8e5b36970bfd82ae715d9e8c3060010000000100000001000202aba1b7d9fc5de9dd93308dc5ebedcb066c8e5b36970bfd82ae715d9e8c306001000000" 
        }
    ],
    "transaction_extensions": [],
    "signatures": [
    "SIG_K1_KY58QhP4jWLJWr7cVkahgL3JAjC8QMK5jnHurFUmn8xU71v6Mh4DmgjY75DxmWE6Je457N6MRM7GapxU43hywnAWKEmC1W"   // 签名 用在 push_transaction 中
    ],
    "context_free_data": []
}
```



**6、http://127.0.0.1:8888/v1/chain/push_transaction （把签名后的交易push 推送到 EOS 系统中，即新建账号完成）**

**请求参数**

|  参数名称   | 参数类型 |                 描述                  |
| :---------: | :------: | :-----------------------------------: |
| compression |  string  |               默认 none               |
|    data     |  string  | abi_json_to_bin 序列化后的 值 binargs |
| signatures  |  string  |           交易签名后的结果            |

**请求示例**

```json
{
  "compression": "none",
  "transaction": {
    "expiration": "2018-05-17T09:54:06.500",
    "ref_block_num": 363759,
    "ref_block_prefix": 4033496171,
    "actions": [
      {
        "account": "eosio",
        "name": "newaccount",
        "authorization": [
          {
            "actor": "bitcoin",
            "permission": "active"
          }
        ],
        "data": "000000603a8ab23b000000ca3d364dfb0100000001000202aba1b7d9fc5de9dd93308dc5ebedcb066c8e5b36970bfd82ae715d9e8c3060010000000100000001000202aba1b7d9fc5de9dd93308dc5ebedcb066c8e5b36970bfd82ae715d9e8c306001000000"    //abi_json_to_bin 的响应参数 binargs
      }
    ]
  },
  "signatures": ["SIG_K1_KY58QhP4jWLJWr7cVkahgL3JAjC8QMK5jnHurFUmn8xU71v6Mh4DmgjY75DxmWE6Je457N6MRM7GapxU43hywnAWKEmC1W"]
}
```

**响应示例**

```json
{
    "transaction_id": "2047702bfdc4678aabe123f335b4b5f604203edf7b4de8e42fa2c9211d4de075",
    "processed": {
        "id": "2047702bfdc4678aabe123f335b4b5f604203edf7b4de8e42fa2c9211d4de075",
        "receipt": {
            "status": "executed",
            "cpu_usage_us": 390,
            "net_usage_words": 25
        },
        "elapsed": 390,
        "net_usage": 200,
        "scheduled": false,
        "action_traces": [
            {
                "receipt": {
                    "receiver": "eosio",
                    "act_digest": "ae18e275184e7defe81be175711cd24206990518963f857715e98755f713957c",
                    "global_sequence": 365444,
                    "recv_sequence": 365419,
                    "auth_sequence": [
                        [
                            "bitcoin",
                            27
                        ]
                    ]
                },
                "act": {
                    "account": "eosio",
                    "name": "newaccount",
                    "authorization": [
                        {
                            "actor": "bitcoin",
                            "permission": "active"
                        }
                    ],
                    "data": {
                        "creator": "bitcoin",
                        "name": "zhangjie",
                        "owner": {
                            "threshold": 1,
                            "keys": [
                                {
                                    "key": "EOS4ufZoTw95yHJS6Cyz3h4w5a2W4cyYpMYRnd7gbFZuCfPxUFS6r",
                                    "weight": 1
                                }
                            ],
                            "accounts": [],
                            "waits": []
                        },
                        "active": {
                            "threshold": 1,
                            "keys": [
                                {
                                    "key": "EOS4ufZoTw95yHJS6Cyz3h4w5a2W4cyYpMYRnd7gbFZuCfPxUFS6r",
                                    "weight": 1
                                }
                            ],
                            "accounts": [],
                            "waits": []
                        }
                    },
                    "hex_data": "000000603a8ab23b000000ca3d364dfb0100000001000202aba1b7d9fc5de9dd93308dc5ebedcb066c8e5b36970bfd82ae715d9e8c3060010000000100000001000202aba1b7d9fc5de9dd93308dc5ebedcb066c8e5b36970bfd82ae715d9e8c306001000000"
                },
                "elapsed": 163,
                "cpu_usage": 0,
                "console": "",
                "total_cpu_usage": 0,
                "trx_id": "2047702bfdc4678aabe123f335b4b5f604203edf7b4de8e42fa2c9211d4de075",
                "inline_traces": []
            }
        ],
        "except": null
    }
}
```

