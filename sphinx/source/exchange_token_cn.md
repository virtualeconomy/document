交易所上Token对接指南
---

这篇文档主要描述了交易所如何上基于VSYS智能合约Euclid发行的Token，完成对Token的充提。

## 基于Euclid发行的Token的简介及特点

Euclid的所有当前contract function都是由封闭的operation code组成，而当前两个可用的token类型的合约里面包含的函数分别为

| Function Index | Token       | Token with Split |
|----------------|-------------|------------------|
| 0              | Supersede   | Supersede        |
| 1              | Issue       | Issue            |
| 2              | Destroy     | Destroy          |
| 3              | Send        | Split            |
| 4              | Transfer    | Send             |
| 5              | Deposit     | Transfer         |
| 6              | Withdraw    | Deposit          |
| 7              | TotalSupply | Withdraw         |
| 8              | MaxSupply   | TotalSupply      |
| 9              | BalanceOf   | MaxSupply        |
| 10             | GetIssuer   | BalanceOf        |
| 11             |             | GetIssuer        |

我们可以看出来，这两种Token主要区别是一种是没有Split功能，而另一种有Split功能，更像是金融市场中的股票，可以拆分/合并Token。

### 关于Split Token的特点

**特点一：** Split是一个在链上完成的操作，经过Split完成拆分/合并Token后，Token Unity发生改变，链上用户拥有的Token Amount（按基本单位算）也会按照比例发生变化。

例如，某Token，基本单位叫DDD，最小单位叫SD，Unity是1000 SD/DDD，Token的基本单位 = 1000 * Token的最小单位，即1 DDD = 1000 SD，则最小Amount为0.001 DDD。当发行方通过Split改变Unity的时候，例如1拆5，Unity变成200，用户原来拥有的1个DDD将会变成5个。

**特点二：** 无论Unity如何变化，换算成最小单位的Amount都不变。

例如，某Token，基本单位叫DDD，最小单位叫SD，Unity是1000 SD/DDD，一个用户在钱包里显示的Token余额是1.00 DDD，换算成最小单位Amount是1000 SD (1.0 DDD * 1000 SD/DDD = 1000 SD)，然后经过Split 1拆5，Unity变成200，用户钱包里显示的Token余额会变成5.00 DDD，换算成最小单位Amount仍然是1000 SD (5.00 DDD * 200 SD/DDD = 1000 SD)。

### Split Token与中心化交易所

既然Split操作只会影响链上数据，那中心化交易所（以下所指的交易所均为“中心化交易所”）应该如何处理带Split的Token的充提数量上的问题呢？下面我们介绍一下交易所Unity拆股协议。

* 首先，交易所上Token前应该要在链上查询并记录Token的Unity，保存在自己的系统中。然后**Token充提均按自己系统保存的Unity换算成最小单位在链上结算**。

* 按照正常流程，Token发行方（项目方）在进行Split前，应通知所有交易所暂停交易和充提，并取消所有的交易挂单。暂停后，Token发行方（项目方）进行Split操作，交易所也更新Token的Unity，再开放交易和充提。

* 对于非正常情况，当出现某些交易所并未通知到位，Split已经完成，但是交易并未停止。因为交易所Token充提均按自己系统保存的Unity换算成最小单位在链上结算，因此充提总价值并没有发生改变，只是用户在链上钱包上看到的会有些不同。（例如，Split操作1拆5，Unity由1000变成200，用户会看到充100个Token到交易所，交易所只显示充了20个；提10个Token到自己钱包的时候，会发现余额增加了50个Token。虽然数量会有差异，但其实充提总价值不发生改变。这个差异直到交易所停止业务、更新Unity后，才会恢复数量同步的显示，用户才会看到一致的余额和充提记录）

## 全节点搭建

参见[全节点搭建教程](https://github.com/virtualeconomy/v-systems/wiki/%E4%BA%A4%E6%98%93%E6%89%80%E5%AF%B9%E6%8E%A5%E6%8C%87%E5%8D%97%28%E4%B8%AD%E6%96%87%29#%E5%90%AF%E5%8A%A8v%E5%85%A8%E8%8A%82%E7%82%B9)

## 全节点接口（API/JSON-RPC）操作


### 方法1：使用SDK（钱包非托管型接入）

我们强烈建议您使用SDK对接链上信息及管理钱包，目前我们提供SDK有如下版本：

JavaScript版SDK [js-v-sdk](https://github.com/virtualeconomy/js-v-sdk)

Java版SDK [java-v-sdk](https://github.com/virtualeconomy/java-v-sdk)

Golang版SDK [go-v-sdk](https://github.com/virtualeconomy/go-v-sdk)

C#版SDK （稍后推出）

### 方法2：使用cURL（钱包托管型接入）

您可以打开浏览器输入```http://<全节点ip>:9922```使用Swagger，在这里也可以查阅所有可以使用的API。

#### 第一步: 准备工作

如果没有安装curl，请安装这个程序，用于发送HTTP请求。

```shell
$ sudo apt-get install curl
```

您可以用以下方法测试连接

```shell
$ curl -X GET --header 'Accept: application/json' 'http://<节点ip>:9922/blocks/height'
```

如果请求成功，您将收到类似这样的回应：

```
{
  "height": 2400326
}
```

#### 第二步: 创建钱包账号及钱包地址查询

参见[创建钱包账号](https://github.com/virtualeconomy/v-systems/wiki/%E4%BA%A4%E6%98%93%E6%89%80%E5%AF%B9%E6%8E%A5%E6%8C%87%E5%8D%97%28%E4%B8%AD%E6%96%87%29#%E7%AC%AC%E4%BA%8C%E6%AD%A5-%E5%88%9B%E5%BB%BA%E9%92%B1%E5%8C%85%E8%B4%A6%E5%8F%B7)及[钱包地址查询](https://github.com/virtualeconomy/v-systems/wiki/%E4%BA%A4%E6%98%93%E6%89%80%E5%AF%B9%E6%8E%A5%E6%8C%87%E5%8D%97%28%E4%B8%AD%E6%96%87%29#%E7%AC%AC%E4%B8%89%E6%AD%A5-%E9%92%B1%E5%8C%85%E5%9C%B0%E5%9D%80%E6%9F%A5%E8%AF%A2%E5%92%8C%E4%BD%99%E9%A2%9D%E6%9F%A5%E8%AF%A2)

#### 第三步: Token及合约信息查询

查询Token信息可以通过 HTTP GET 调用 /contract/tokenInfo/{tokenId}

```shell
$ curl -X GET 'http://<节点ip>:9922/contract/tokenInfo/TWsVx4qkjs2iPTw8zrhcKq1FDe1ygzhDEwiFY4ap7'
```

如果成功将返回类似结果:

```
{
  "tokenId": "TWsVx4qkjs2iPTw8zrhcKq1FDe1ygzhDEwiFY4ap7",
  "contractId": "CEsHcSQdNbjHv9q8sPaEGKyDCJ243s6JYoQ",
  "max": 2100000000000000,
  "total": 11000000000000,
  "unity": 100000000,
  "description": "1TXtSrHmSLPYFxXWRkEs4BQscUFb6yaPfP7RTpR31Rk"
}
```

返回中， `max`和`total`均为最小单位，`max`表示Token发行的最大量为21000000个Token，`total`表示已发行的流通量为110000个Token，`Unity`100000000表示此Token最小Amount为0.00000001。`description`为Base58编码的字符串，是该Token的描述。


查询合约信息可以通过 HTTP GET 调用 /contract/info/{contractId}

```shell
$ curl -X GET 'http://<节点ip>:9922/contract/info/CEsHcSQdNbjHv9q8sPaEGKyDCJ243s6JYoQ'
```

如果成功将返回类似结果:

```
{
  "contractId": "CEsHcSQdNbjHv9q8sPaEGKyDCJ243s6JYoQ",
  "transactionId": "HcXbHTkde1qg7H313Wgzg3j7GC2Dk74mm6JwPvSykrvD",
  "type": "TokenContract",
  "info": [
    {
      "data": "AUCvVbLeLU5LiLxyYgEYcnccUDxJuk3JMeA",
      "type": "Address",
      "name": "issuer"
    },
    {
      "data": "AUCvVbLeLU5LiLxyYgEYcnccUDxJuk3JMeA",
      "type": "Address",
      "name": "maker"
    }
  ],
  "height": 6616970
}
```

返回中， `transactionId`是创建合约的交易ID，`type`是合约的类型，如果是带Split功能的Token合约为TokenContractWithSplit，如果是不带Split功能的Token合约为TokenContract。Token合约的`info`中包含了2个地址信息，一个是Issuer，管理增发、销毁、拆/合Token的钱包地址；另一个是Maker，创建合约的钱包地址，Maker可以通过调用合约Supersede函数来变更Issuer。`height`是创建合约的交易所在区块高度。

#### 第四步: Token余额查询

查询余额可以通过 HTTP GET 调用 /contract/balance/{address}/{tokenId}

```shell
$ curl -X GET 'http://<节点ip>:9922/contract/balance/AU8R9ri7eG968zuJuLQVLMiUzRNXvQwNPwE/TWsVx4qkjs2iPTw8zrhcKq1FDe1ygzhDEwiFY4ap7'
```

如果成功将返回类似结果:

```
{
  "address": "AU8R9ri7eG968zuJuLQVLMiUzRNXvQwNPwE",
  "tokenId": "TWsVx4qkjs2iPTw8zrhcKq1FDe1ygzhDEwiFY4ap7",
  "balance": 1234400000,
  "unity": 100000000
}
```

返回中， `balance`为换算成最小单位的Amount，1234400000表示用户拥有1234.4个Token

#### 第四步: 发送Token

如果用户提Token到自己钱包，交易所可使用 HTTP POST 调用 /contract/execute 完成Token发送

```shell
$ curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' --header 'api_key: <节点api密钥>' -d '{ \ 
   "sender": "ATtRykARbyJS1RwNsA6Rn1Um3S7FuVSovHK", \
   "contractId": "CF8eRsiyXSP9qnrby9AqgzJGJbjYx9aXkbi", \ 
   "functionIndex": 4, \ 
   "functionData": "14uNyNb5nFsicZ6dELMKjNAHz3PLM3mjiu6pHsbTrgTYYWb5WRy", \ 
   "attachment": "", \ 
   "fee": 30000000, \ 
   "feeScale": 100 \ 
 }' 'http://<节点ip>:9922/vsys/payment'
```

在请求的JSON结构中，

```contractId``` 是合约ID，可以通过上文第三步获取得到。

```functionIndex``` 是执行函数序号。如果是带Split功能的Token，此处应填4；如果是不带Split功能的Token，此处应填3。

```fee``` 是交易费，目前是固定交易费30000000 (0.3 VSYS)。

```feeScale ``` 目前是固定值100。

```sender ``` 是支付方的钱包地址。

```attachment ``` 可以是任意字符（最多140个字符），用base58编码填入此项。

```functionData```是传入参数然后组合起来，再用Base58编码。

**正确编码functionData传参方法：**

Send Token这个函数在合约里是2个参数，第一个是收Token人的地址，第二个是数量

```
function send(Account recipient, Amount amount)
```

而参数（我们称作DataEntry）的Type ID如下：

```
PublicKey = 1
Address = 2
Amount = 3
Int32 = 4
ShortText = 5
ContractAccount = 6
```

除了ShortText类型，其他类型的DataEntry编码方式为Type ID + Byte Value。

例如我们发送1.5个Token给AU1L8NVL9NTvu4n4XbP3fPJ8b5gQGjRG17W，此Token的Unity为1000。我们先把地址用base58解码得到26个Bytes：

```
(Base58 decode AU1L8NVL9NTvu4n4XbP3fPJ8b5gQGjRG17W)
05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1
```

然后我们把Token Amount换成最小单位，得到的整数转成8个Bytes：

```
(1.5 * 1000 = 1500)
00 00 00 00 00 00 05 dc
```

所以我们得到这些Bytes

```
00 02（参数个数）
02（第一个参数的类型ID）
05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1（第一个参数的值）
03（第二个参数的类型ID）
00 00 00 00 00 00 05 dc（第二个参数的值）
```

然后我们把这些Bytes拼起来

```
00 02 02 05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1 03 00 00 00 00 00 00 05 dc
```

再用base58编码，得到

```
14uNyNb5nFsicZ6dELMKjNAHz3PLM3mjiu6pHsbTrgTYYWb5WRy
```

如果成功将返回类似结果:

```
{
  "type": 9,
  "id": "FZcxfa6oD9VumAseRnDx7bUWeMmbs8c4R9TreEhRwNh3",
  "fee": 30000000,
  "timestamp": 1568346104946082600,
  "proofs": [
    {
      "proofType": "Curve25519",
      "publicKey": "DREQ2s3dQzaKTsvnmKWTnj59ptM1NzXwmap8jYVdPeLC",
      "address": "AUCJE7djDBbFrgeKRZ4UMREujAeqnD6L6Ph",
      "signature": "3NPGPyxWW8PDh1Hc3hubZcRPk3KJBkKsjidTVaFR32Jv1obLfBAxEY5Vq27ZNzfPqcvvi64QtaKcZTXZMxbjXGqx"
    }
  ],
  "contractId": "CF8eRsiyXSP9qnrby9AqgzJGJbjYx9aXkbi",
  "functionIndex": 4,
  "functionData": "14uNyNb5nFsicZ6dELMKjNAHz3PLM3mjiu6pHsbTrgTYYWb5WRy",
  "attachment": "",
  "feeScale": 100
}
```

#### 第五步: 查询交易记录

如果一笔交易已经打包到区块中，通过`GET /transactions/info/{id}`这个API可以得到确切的知道该笔交易所在区块高度。当节点同步高度高于该交易的所在高度31个区块之后，可确认该笔交易。

**解析functionData得到收Token的地址和Amount的方法：**

例如，我们解析一个Send函数的functionData

```
function send(Account recipient, Amount amount)
```

我们在链上交易得到的functionData值为

```
14uNyNb5nFsicZ6dELMKjNAHz3PLM3mjiu6pHsbTrgTYYWb5WRy
```

用base58解码，得到

```
00 02 02 05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1 03 00 00 00 00 00 00 05 dc
```

Address类型固定长度26个Bytes，Amount类型固定长度8个Bytes

```
00 02（参数个数）
02（第一个参数的类型ID）
05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1（第一个参数的值）
03（第二个参数的类型ID）
00 00 00 00 00 00 05 dc（第二个参数的值）
```

根据第一个参数的值，我们用得到Bytes Base58编码拿到收Token的地址AU1L8NVL9NTvu4n4XbP3fPJ8b5gQGjRG17W

根据第二个参数的值，把Bytes转化成64位整数，得到1500。如果Token的Unity是1000，则表示这个地址收到了1.5个Token。

几个常见的交易类型ID:

```
2 = 支付交易
3 = 租赁交易
4 = 取消租赁交易
5 = 铸币交易
8 = 注册合约交易
9 = 执行合约函数交易
```

##### 1.通过钱包地址查询交易记录
由于节点只能缓存最近10000条记录，中心化交易所应监控交易类型为9（执行合约函数类型交易），Contract ID为指定值的，且Function Index为3（不带Split的Token）或者4（带Split的Token）的交易记录，并存入自己的数据库，此类型记录为Token转账记录，可以方便今后用户查询Token充提时调用。

用 HTTP GET 调用 /transactions/address/{address}/limit/{limit} API，例如，

```shell
$ curl -X GET 'http://<节点ip>:9922/transactions/address/ATt6P4vSpBvBTHdV5V9PJEHMFp4msJ1fkkX/limit/10'
```

如果成功将返回类似结果:

```
[
  [
     {
      "type": 9,
      "id": "FZcxfa6oD9VumAseRnDx7bUWeMmbs8c4R9TreEhRwNh3",
      "fee": 30000000,
      "timestamp": 1568346104946082600,
      "proofs": [
        {
          "proofType": "Curve25519",
          "publicKey": "DREQ2s3dQzaKTsvnmKWTnj59ptM1NzXwmap8jYVdPeLC",
          "address": "AUCJE7djDBbFrgeKRZ4UMREujAeqnD6L6Ph",
          "signature": "3NPGPyxWW8PDh1Hc3hubZcRPk3KJBkKsjidTVaFR32Jv1obLfBAxEY5Vq27ZNzfPqcvvi64QtaKcZTXZMxbjXGqx"
        }
      ],
      "contractId": "CF8eRsiyXSP9qnrby9AqgzJGJbjYx9aXkbi",
      "functionIndex": 4,
      "functionData": "14uNyNb5nFsicZ6dELMKjNAHz3PLM3mjiu6pHsbTrgTYYWb5WRy",
      "attachment": "",
      "feeScale": 100,
      "status": "Success",
      "feeCharged": 30000000,
      "height": 7257386
    },
    {
      "type": 2,
      "id": "7YXd42MiPyyUfVmvVyV2b3c6uL3FgtrZTRbHe3sWydC8",
      "fee": 10000000,
      "timestamp": 1568256953919000000,
      "proofs": [
        {
          "proofType": "Curve25519",
          "publicKey": "2wMSwLfwoPTt2BpsTZJ6djd2W92BwYR8nkGjBD4aAae3",
          "address": "AU1L8NVL9NTvu4n4XbP3fPJ8b5gQGjRG17W",
          "signature": "2Q5i6BRjTTErRqWDmznWdd7JKMQERs329fZ4cqj2KAXbVot9szaSLzFsnrmSonhvaHazCmYChkEeSXry35xKmvF3"
        }
      ],
      "recipient": "AUCJE7djDBbFrgeKRZ4UMREujAeqnD6L6Ph",
      "feeScale": 100,
      "amount": 10000000000,
      "attachment": "3A836b",
      "status": "Success",
      "feeCharged": 10000000,
      "height": 7232281
    },
    ...
  ]
]
```
##### 2.通过区块来查询交易记录
用 HTTP GET 调用 /blocks/seq/{from}/{to} API，例如，

```shell
$ curl -X GET 'http://<节点ip>:9922/blocks/seq/10/15'
```

如果成功将返回类似结果:

```
[
  {
    "version": 1,
    "timestamp": 1544759701000897500,
    "reference": "3Mbnc1bqdWP2AkjWbRxgqkQRJYTVQUxvZApCj1fJT96CJbKs5VuBHQ4JZepunQ1CnY4GAJHSNgxFntT37FCikFZd",
    "SPOSConsensus": {
      "mintTime": 1544759701000000000,
      "mintBalance": 18517768509775
    },
    "resourcePricingData": {
      "computation": 0,
      "storage": 0,
      "memory": 0,
      "randomIO": 0,
      "sequentialIO": 0
    },
    "TransactionMerkleRoot": "AtsZrQaqaBjQupNyJbU819dzkQWTsD1xGRdtkcDd2XYu",
    "transactions": [
      {
        "type": 2,
        "id": "BCq1Ai1ZjWWCRr1ZZSdz4EeZsjy5UfJM7fhnxzrePQNu",
        "fee": 10000000,
        "timestamp": 1544759672378309000,
        "proofs": [
          {
            "proofType": "Curve25519",
            "publicKey": "3orvgyRKf45FRyiCkcA3CzAGDvyEpBpXZzYGEGZnpZK5",
            "address": "ATtRykARbyJS1RwNsA6Rn1Um3S7FuVSovHK",
            "signature": "2bQ9H7fX6HvdDgzMDKb3Pz3U2xf66Ffq5QwU4kJLfWvu9JqrnCArqubKCRENLt9nDbWGQNdkgQRiUTcVPgo7gVYK"
          }
        ],
        "recipient": "AUD9poQ3ernHVx2kUz9XRJvmCJEk6sZrj9T",
        "feeScale": 100,
        "amount": 5000000000000,
        "attachment": "",
        "status": "Success",
        "feeCharged": 10000000
      },
      {
        "type": 5,
        "id": "J7TaTi3YPEiUUa4dch3HfW4FhEct5dFw9LUaWZSE8Vyw",
        "recipient": "ATtRykARbyJS1RwNsA6Rn1Um3S7FuVSovHK",
        "timestamp": 1544759701000897500,
        "amount": 900000000,
        "currentBlockHeight": 10,
        "status": "Success",
        "feeCharged": 0
      }
    ],
    "generator": "ATtRykARbyJS1RwNsA6Rn1Um3S7FuVSovHK",
    "signature": "44Ub1bUBzBtDMiwkDqomaa6w8J9iAYYXW98jkGY8YiRZHV3EUUTeEpeiBpsSAzQE7X6B8QhiU2vP2UEs82Zb1GzV",
    "fee": 10000000,
    "blocksize": 500,
    "height": 10,
    "transaction count": 2
  }
  ...
]
```

##### 3.通过交易ID查询交易记录
用 HTTP GET 调用 /transactions/info/{id} API，例如，

```shell
$ curl -X GET 'http://<节点ip>:9922/transactions/info/FZcxfa6oD9VumAseRnDx7bUWeMmbs8c4R9TreEhRwNh3'
```

如果成功将返回类似结果:

```
{
  "type": 9,
  "id": "FZcxfa6oD9VumAseRnDx7bUWeMmbs8c4R9TreEhRwNh3",
  "fee": 30000000,
  "timestamp": 1568346104946082600,
  "proofs": [
    {
      "proofType": "Curve25519",
      "publicKey": "DREQ2s3dQzaKTsvnmKWTnj59ptM1NzXwmap8jYVdPeLC",
      "address": "AUCJE7djDBbFrgeKRZ4UMREujAeqnD6L6Ph",
      "signature": "3NPGPyxWW8PDh1Hc3hubZcRPk3KJBkKsjidTVaFR32Jv1obLfBAxEY5Vq27ZNzfPqcvvi64QtaKcZTXZMxbjXGqx"
    }
  ],
  "contractId": "CF8eRsiyXSP9qnrby9AqgzJGJbjYx9aXkbi",
  "functionIndex": 4,
  "functionData": "14uNyNb5nFsicZ6dELMKjNAHz3PLM3mjiu6pHsbTrgTYYWb5WRy",
  "attachment": "",
  "feeScale": 100,
  "status": "Success",
  "feeCharged": 30000000,
  "height": 7257386
}
```

### 冷钱包支付Token签名

为了资产安全，一般交易所会把用户账号上的热钱包的Token转移到冷钱包存储，当用户提Token的时候再把冷钱包的Token转出来给用户。

当然，您也可以编写自己的程序生成冷钱包签名，我们一步一步解析生成签名的步骤。

例如，我们希望冷钱包发起如下JSON参数的交易：

```
{
  "fee": 30000000,
  "feeScale": 100,
  "timestamp": 1547722056762119200,
  "contractId": "CF8eRsiyXSP9qnrby9AqgzJGJbjYx9aXkbi",
  "functionIndex": 4,
  "functionData": "14uNyNb5nFsicZ6dELMKjNAHz3PLM3mjiu6pHsbTrgTYYWb5WRy",
  "senderPublicKey": "B2Khd89jtnpuzGdnyGRcnKycZMBCo6PsotFcWWi1wMDV",
  "attachment": "HXRC"
}
```

把这些参数项按如下顺序转化成Bytes：

```
type_id: 09
contract_id: 06 54 b0 b1 ac b4 bd 2c db 7d 04 9f 96 e0 0b a7 a6 9e cb da 81 b9 72 6b b0 e9
function_index: 00 04
function_data: 00 02 02 05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1 03 00 00 00 00 00 00 05 dc
length of attachment: 00 03
attachment: 31 32 33
tx_fee: 00 00 00 00 01 c9 c3 80
fee_scale: 00 64
timestamp: 15 7a 9d 02 ac 57 d4 20
```

然后拼接起来：

```
09 06 54 b0 b1 ac b4 bd 2c db 7d 04 9f 96 e0 0b a7 a6 9e cb da 81 b9 72 6b b0 e9 00 04 00 02 02 05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1 03 00 00 00 00 00 00 05 dc 00 03 31 32 33 00 00 00 00 01 c9 c3 80 00 64 15 7a 9d 02 ac 57 d4 20
```

最终，我们用冷钱包私钥通过[curve25519](https://github.com/tgalal/python-axolotl-curve25519)的ed25519方法来完成签名。

```
(For reference only. The signature will be different if generate again)
72 74 61 73 6d 50 31 4c 63 48 79 5a 63 71 35 36 67 52 34 78 57 45 35 78 68 54 78 59 35 33 6f 6f 6f 4d 32 53 61 36 63 75 42 52 61 78 72 71 33 39 63 54 56 6b 39 4d 67 4d 76 6e 38 61 5a 45 6d 4e 78 6b 56 39 55 39 63 62 41 6a 43 50 4d 68 48 46 6f 51 33 57 69 66 57
```

把得到的签名用base58编码，然后放到JSON的`signature`项。

```
{
  "fee": 30000000,
  "feeScale": 100,
  "timestamp": 1547722056762119200,
  "contractId": "CF8eRsiyXSP9qnrby9AqgzJGJbjYx9aXkbi",
  "functionIndex": 4,
  "functionData": "14uNyNb5nFsicZ6dELMKjNAHz3PLM3mjiu6pHsbTrgTYYWb5WRy",
  "senderPublicKey": "B2Khd89jtnpuzGdnyGRcnKycZMBCo6PsotFcWWi1wMDV",
  "attachment": "HXRC"
  "signature": "rtasmP1LcHyZcq56gR4xWE5xhTxY53oooM2Sa6cuBRaxrq39cTVk9MgMvn8aZEmNxkV9U9cbAjCPMhHFoQ3WifW"
}
```

将这个JSON传递给全节点，然后全节点用API`/contract/broadcast/execute`广播到网络上。