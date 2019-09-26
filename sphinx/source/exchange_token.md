Instructions of Listing Token for Exchanges
---

The purpose of this document is to describe how to list VSYS smart contract (Euclid) based Token for Exchanges, including deposit and withdraw token.

## Euclid based Token Introduction

All of contract functions in Euclid consists of sealed operation code. Currnetly, there is two kinds of token contract is available on blockchain. The contract functions are as the following:

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

We can see the different between these two token is one contains Split function while another not. The token contract with split function likes stock in financal market. We can split/revert-split token by this function.

### About Split Token Feature

**Feature 1:** Split is an operation which impacts on blockchain. After split/revert-split token, the token Unity will be changed. The amount of token that user held on blockchain will also be changed accordingly (calculated in regular unit).

For example, one token, regular unit named DDD, mininum unit named SD, unity is 1000 SD/DDD. So 1 DDD = 1000 SD. The mininum amount will be 0.001 DDD. Bob held 1.0 DDD. After token issuer set new Unity to 200, 1 token split into 5. When Bob check token balance in wallet, he will found he held 5.0 DDD.

**Feature 2:** Whatever Unity is, the amount calculated in mininum unit will not be changed.

For example, one token, regular unit named DDD, mininum unit named SD, unity is 1000 SD/DDD. Bob held 1.0 DDD. Calculated in mininum unit, it is 1000 SD (1.0 DDD * 1000 SD/DDD = 1000 SD). After token issuer set new Unity to 200, 1 token split into 5. When Bob check token balance in wallet, he will found he held 5.0 DDD. Calculated in mininum unit, it is still 1000 SD (5.0 DDD * 200 SD/DDD = 1000 SD).

### How Centralized Exchange handles Split Token

Split is an operation which impacts on blockchain. So how centralized exchange handles split token deposit and withdraw? Let us introduce the Exchange Unity Split/Revert-split Protocol.

* First, exchange should check and store the token unity on blockchain via API/JSON-RPC. **All Token deposit and withdraw amount should calculate in mininum unit, and then broadcast the transaction on blockchain.**

* In regular Split workflow, Token Issuer should notice exchange that they will split token at some time. The exchange will stop all business (trading, deposit, withdraw and cancel all orders) of this token before split. Then Issuer does split operation and exchange updates the unity. Finally, exchange resumes all business of this token.

* In some abnormal case, exchange somehow did not get notice from token issue and business did not stop while token splitting. For exchange use self-store unity to calculate the amount of deposit and withdraw in mininum unit, the total value of Token sent on chain is the same. Only they will be different amount (in regular unit display) between exchange and his/her wallet. (For example, token issuer set new Unity to 200 from 1000. When user deposit 100 Token to exchange, he/she will see just 20 Token displayed at exchange; If he/she withdraw 10 Token from exchange to his/her own wallet, he/she will see 50 Token transfered from exchange. Though the regular unit amout is different, the total value of deposit and withdraw is the same. The abnormal display will keep showing until exchange stop business and update new unity. Once done, user will see consistent balance and transaction amount in exchange and his/her own wallet.)

## Full Node Setup

Please refer [Full Node Setup Instructions](https://github.com/virtualeconomy/v-systems/wiki/Instructions-for-Exchanges#start-v-systems-full-node)

## Full Node API/JSON-RPC Operation


### Method 1: Use SDK (Independent Wallet Integration)

We strongly suggest you use SDK to do integration. Currently, we provide the following version of SDK:

JavaScript version SDK [js-v-sdk](https://github.com/virtualeconomy/js-v-sdk)

Java version SDK [java-v-sdk](https://github.com/virtualeconomy/java-v-sdk)

Golang version SDK [go-v-sdk](https://github.com/virtualeconomy/go-v-sdk)

C# version SDK (Work in progress)

### Method 2: Use cURL (Node Delegate Wallet Integration)

You could open ```http://<full node ip>:9922``` in browser to find all APIs you can use.

#### Step 1: Prepare

Install curl for post http request if you do not have curl.

```shell
$ sudo apt-get install curl
```

To test connection, you could use

```shell
$ curl -X GET 'http://<full node ip>:9922/blocks/height'
```

If success, you will get response more or less like the following:

```
{
  "height": 2400326
}
```

#### Step 2: Create Wallet and Check Address in Node

Please refer [Create Wallet](https://github.com/virtualeconomy/v-systems/wiki/Instructions-for-Exchanges#step-2-create-wallet) and [Check Address in Node](https://github.com/virtualeconomy/v-systems/wiki/Instructions-for-Exchanges#step-3-check-address-and-balance)

#### Step 3: Check Token Contract Info

We can check Token info via HTTP GET /contract/tokenInfo/{tokenId}

```shell
$ curl -X GET 'http://<full node ip>:9922/contract/tokenInfo/TWsVx4qkjs2iPTw8zrhcKq1FDe1ygzhDEwiFY4ap7'
```

If success, you will get response more or less like the following:

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

In response, `max` and `total` are returned as minimum unity amout. `max` means the max supply of amount is 21000000.0 Token. `total` means the 
circulating supply is 110000.0 Token. `unity` = 100000000 means the minimum amount of this Token is 0.00000001. `description` is a string which encoded by Base58.


We can check Contract info via HTTP GET /contract/info/{contractId}

```shell
$ curl -X GET 'http://<full node ip>:9922/contract/info/CEsHcSQdNbjHv9q8sPaEGKyDCJ243s6JYoQ'
```

If success, you will get response more or less like the following:

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

In response, `transactionId` is the transaction ID that contract created. `type` is contract type description. If it is token contract which contains "Split" function, the value of this field will be "TokenContractWithSplit". If it is token contract which does not contain "Split" function, the value of this field will be "TokenContract". Token `info` contains 2 wallet address: one is Issuer, which manages issue, destroy, split action; another is Maker, which is creator of this contract. Maker can set Issuer by Supersede function. `height` shows the contract is created in which block height.

#### Step 4: Check Token Balance

We can check Token balance via HTTP GET /contract/balance/{address}/{tokenId}

```shell
$ curl -X GET 'http://<full node ip>:9922/contract/balance/AU8R9ri7eG968zuJuLQVLMiUzRNXvQwNPwE/TWsVx4qkjs2iPTw8zrhcKq1FDe1ygzhDEwiFY4ap7'
```

If success, you will get response more or less like the following:

```
{
  "address": "AU8R9ri7eG968zuJuLQVLMiUzRNXvQwNPwE",
  "tokenId": "TWsVx4qkjs2iPTw8zrhcKq1FDe1ygzhDEwiFY4ap7",
  "balance": 1234400000,
  "unity": 100000000
}
```

In response, `balance` is show as the mininum unity amount. For example, 1234400000 means the user holds 1234.4 Token.

#### Step 5: Send Token

(Sending Token is a complex progress. We suggest you use SDK to send token)

If user withdraws Token to his/her own wallet, the exchange can call HTTP POST /contract/execute to send Token.

```shell
$ curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' --header 'api_key: <api_key>' -d '{ \ 
   "sender": "ATtRykARbyJS1RwNsA6Rn1Um3S7FuVSovHK", \
   "contractId": "CF8eRsiyXSP9qnrby9AqgzJGJbjYx9aXkbi", \ 
   "functionIndex": 4, \ 
   "functionData": "14uNyNb5nFsicZ6dELMKjNAHz3PLM3mjiu6pHsbTrgTYYWb5WRy", \ 
   "attachment": "", \ 
   "fee": 30000000, \ 
   "feeScale": 100 \ 
 }' 'http://<full node ip>:9922/vsys/payment'
```

In POST JSON response,

```contractId``` is contract ID which we get in step 3.

```functionIndex``` is the index of executed function. For token contract with split function, the index here is 4; for token contract that no split function, the index here is 3.

```fee``` is transaction fee. Currently it is constant value 30000000 (0.3 VSYS).

```feeScale ``` Currently it is constant value 100.

```sender ``` is wallet address of sender.

```attachment ``` could be any text (max 140 characters) with base58 encoding.

```functionData``` is packaged function parameters with base58 encoding.

**The instruction of encoding functionData:**

The contract function of "Send Token" contains two parameters. The first parameter is recipient address. The second one is amount of sending token.

```
function send(Account recipient, Amount amount)
```

And Type ID of parameter (we called DataEntry) is as the following:

```
PublicKey = 1
Address = 2
Amount = 3
Int32 = 4
ShortText = 5
ContractAccount = 6
```

Except ShortText type, one DataEntry encodes with Type ID + Byte Value.

For example, if we want to send 1.5 Token to AU1L8NVL9NTvu4n4XbP3fPJ8b5gQGjRG17W. The token unity is 1000. First, we decode the address with Base58 and get 26 bytes:

```
(Base58 decode AU1L8NVL9NTvu4n4XbP3fPJ8b5gQGjRG17W)
05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1
```

And then we calculate the amount in mininum unit and convert to 8 bytes:

```
(1.5 * 1000 = 1500, convert 1500 to 8 bytes)
00 00 00 00 00 00 05 dc
```

So we can get these bytes:

```
00 02 (The num of parameters)
02 (The type ID of the first parameter)
05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1 (The value of the first parameter)
03 (The type ID of the second parameter)
00 00 00 00 00 00 05 dc (The value of the second parameter)
```

Then we concat these bytes

```
00 02 02 05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1 03 00 00 00 00 00 00 05 dc
```

Finally, encode with Base58, we get

```
14uNyNb5nFsicZ6dELMKjNAHz3PLM3mjiu6pHsbTrgTYYWb5WRy
```

If success, you will get response more or less like the following:

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

#### Step 6: Check transaction record

To simplify contract parse on chain, we also provide token transaction query node for exchange to check the token transaction history. Exchange can parse contract by itself as well. For parse instructions, please refer [here](https://github.com/virtualeconomy/v-systems/wiki/Instructions-of-Listing-Token-for-Exchanges/b3a3fbcd7edb3249c9775170e1c280c9a057029e#step-5-check-transaction-record)

##### 1.Get Transaction History by Address

Use HTTP GET /token/transactions?tokenId={tokenId}&address={address}&limit={limit}&start={startIndex} API. `start` is optional. In reponse for each query, you will get LastEvaluatedKey, which you can use it as start input value for next paging query. For example,

```shell
$ curl -X GET 'https://token-explorer-dev.vcloud.systems/api/v1/token/transactions?tokenId= TWZ11xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxhx1wSY&address=ARMMfCxxxxxxxxxxxxxxxxxxxxxxxiAZFSS&limit=10
```

If success, you will get response more or less like the following:

```
{
    "address": "address_in_query_parameter_or_null",
    "limit": "limit_in_query_parameter_or_default_10",
    "result": {
        "Count": 1,
        "Items": [
            {
                "confirmations": 6700,
                "content": {
                    "data": {
                        "amount": 500000000000000,
                        "amountStr": "500000000000000",
                        "recipient": "ARBGvExxxxxxxxxxxxxxxxxxxxxxaMD6yyP"
                    },
                    "functionIndex": 3
                },
                "contractId": "CC3KZzxxxxxxxxxxxxxxxxxxxxxxxpEPfnM",
                "createdAt": 0,
                "functionIndex": 3,
                "functionIndexType": "send",
                "height": 6442672,
                "signer": "ARMMfCxxxxxxxxxxxxxxxxxxxxxxxiAZFSS",
                "status": "Success",
                "timestamp": 1560000000000,
                "timestampStr": "1560000000000000",
                "tokenId": "TWZ11xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxhx1wSY",
                "transactionFee": 0.3,
                "transactionId": "8UJzh7xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxWbp4YB",
                "updatedAt": 0,
                "verifications": 1
            }
        ],
        "LastEvaluatedKey": 1569395070031
    },
    "tokenId": "tokenId_in_query_parameter"
}
```


##### 2.Get Transaction History by Block Height

Use HTTP GET /token/transactions?tokenId={tokenId}&startHeight={from}&endHeight={to}&limit={limit}&start={startIndex} API

```shell
$ curl -X GET 'https://token-explorer-dev.vcloud.systems/api/v1/token/transactions?tokenId= TWZ11xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxhx1wSY&limit=10&startHeight=6100000&endHeight=6100100'
```

If success, you will get response more or less like the following:

```
{
    "address": "address_in_query_parameter_or_null",
    "limit": "limit_in_query_parameter_or_default_10",
    "result": {
        "Count": 1,
        "Items": [
            {
                "confirmations": 6700,
                "content": {
                    "data": {
                        "amount": 500000000000000,
                        "amountStr": "500000000000000",
                        "recipient": "ARBGvExxxxxxxxxxxxxxxxxxxxxxaMD6yyP"
                    },
                    "functionIndex": 3
                },
                "contractId": "CC3KZzxxxxxxxxxxxxxxxxxxxxxxxpEPfnM",
                "createdAt": 0,
                "functionIndex": 3,
                "functionIndexType": "send",
                "height": 6442672,
                "signer": "ARMMfCxxxxxxxxxxxxxxxxxxxxxxxiAZFSS",
                "status": "Success",
                "timestamp": 1560000000000,
                "timestampStr": "1560000000000000",
                "tokenId": "TWZ11xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxhx1wSY",
                "transactionFee": 0.3,
                "transactionId": "8UJzh7xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxWbp4YB",
                "updatedAt": 0,
                "verifications": 1
            }
        ],
        "LastEvaluatedKey": 1569395070031
    },
    "tokenId": "tokenId_in_query_parameter"
}
```


##### 3.Get Transaction by ID

Use HTTP GET /transactions/info/{id} API

```shell
$ curl -X GET 'https://token-explorer-dev.vcloud.systems/api/v1/token/transaction/HNvRwxxxxxxxxxxxxxxxxxxxxxxxxxxxxxVU9N4o'
```

If success, you will get response more or less like the following:

```
{
    "result": {
        "confirmations": 6800,
        "content": {
            "data": {
                "amount": 500000000000000,
                "amountStr": "500000000000000",
                "recipient": "ARMMfCxxxxxxxxxxxxxxxxxxxxxxxxiAZFAA"
            },
            "functionIndex": 3
        },
        "contractId": "CC3KZzxxxxxxxxxxxxxxxxxxxxxxxxEPfasd",
        "createdAt": 0,
        "functionIndex": 3,
        "functionIndexType": "send",
        "height": 6440000,
        "signer": "ARBGvExxxxxxxxxxxxxxxxxxxxxxxxMD1yyP",
        "status": "Success",
        "timestamp": 1569390000000,
        "timestampStr": "1569390000000000000",
        "tokenId": "TWZ11xxxxxxxxxxxxxxxxxxxxxxxxxxxxxhx3wSY",
        "transactionFee": 0.3,
        "transactionId": "HNvRwxxxxxxxxxxxxxxxxxxxxxxxxxxxxxVU9N4o",
        "updatedAt": 0,
        "verifications": 1
    }
}
```


#### Step 7: Check block height

Use HTTP GET /blocks/height API

```shell
$ curl -X GET 'http://<full node ip>:9922/blocks/height'
```

Response:

```
{
  "height": 6145911
}
```

If a transaction if packaged into a block, the API `GET /transactions/info/{id}` will return the **height** of transaction. We can confirm a transaction if node block height is 31 blocks higher than the height of transaction.

Some common id of transaction types:

```
2 = Payment transaction
3 = Lease transaction
4 = Cancel lease transaction
5 = Minting transaction
8 = Register contract transaction
9 = Execute contract function transaction
```

### Cold Wallet Signature for Sending Token

(Sending Token is a complex progress. We suggest you use SDK to send token)

To make sure asset is safty, the exchange collects Token from hot wallet and stores these Token to cold wallet. When user widthdraw Token, the exchange transfer out to user from cold wallet. 

And you can write your own program to generate Token payment signature as well. For example, if you want to send Token which JSON format like this:

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

Convert these fields into bytes:

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

And then combine togather:

```
09 06 54 b0 b1 ac b4 bd 2c db 7d 04 9f 96 e0 0b a7 a6 9e cb da 81 b9 72 6b b0 e9 00 04 00 02 02 05 54 66 33 43 4d 30 9c 92 cf 07 47 da 4b 40 53 15 22 60 c1 e8 4f 62 05 dd d1 03 00 00 00 00 00 00 05 dc 00 03 31 32 33 00 00 00 00 01 c9 c3 80 00 64 15 7a 9d 02 ac 57 d4 20
```

Finally, we used ed25519 of [curve25519](https://github.com/tgalal/python-axolotl-curve25519) library to signature.

```
(For reference only. The signature will be different if generate again)
72 74 61 73 6d 50 31 4c 63 48 79 5a 63 71 35 36 67 52 34 78 57 45 35 78 68 54 78 59 35 33 6f 6f 6f 4d 32 53 61 36 63 75 42 52 61 78 72 71 33 39 63 54 56 6b 39 4d 67 4d 76 6e 38 61 5a 45 6d 4e 78 6b 56 39 55 39 63 62 41 6a 43 50 4d 68 48 46 6f 51 33 57 69 66 57
```

And then encode with base58 and put in `signature` field.

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

Pass this JSON to full node. And full node broadcasts to network with API `/contract/broadcast/execute`.