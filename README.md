# MixSwap 开发文档
MixSwap 是 Exin 旗下的 MiFi DEX 聚合交易平台，它同时接入多个交易所的深度，目标是帮助用户兑换更多的币，省更多的钱。

## 交易

要发起交易，需要将待交易的币以指定的 Memo 格式转账给 MixSwap 机器人 (Mixin ID: 7000103767 User ID: 6a4a121d-9673-4a7e-a93e-cb9ca4bb83a2)

### 用户转账 Memo 规范

每个字段用 `|` 隔开，并以 BASE64 编码：

ACTION|FIELD1|FIELD2|FIELD3

| 行为 | ACTION | FIELD1 | FIELD2 | FIELD3 |
| ---- | ---- | ---- | ----| ---- |
| 聚合交易 | 0 | 目标资产UUID | Route ID。可选。 | 最少获取数量。可选。 |

其中：
* Route ID 是一个代表这笔交易的路线分布的字符串，可以通过接口获取。
  
  不填或者填0的话，MixSwap 会在收到转账后自动计算当前最优交易分布。

* 最小获取数量 是允许成交的最小所得数量。

  如果成交结果少于该数量，会交易失败自动退币。

  不填的话会按照千分之二的最大滑点持续成交直至将支付的币全部交易完；

* **初次接入时，建议小额测试。**

  **目前对于不符合规范（不能正确解码、未用`|`分隔、ACTION 不正确等）的转账不会自动退币。**


### 服务器转账 Memo 规范
每个字段会以 `|` 隔开，并以 BASE64 编码：

RESULT|TRACE|SOURCE|TYPE

| 字段 | RESULT | TRACE | SOURCE | TYPE |
| ---- | ---- | ---- | ---- | ---- |
| 参数校验失败退币 | 1 | 用户支付的Trace ID | SN (SNapshot) | RF (RFund) |
| 聚合交易 | 成功 0 / 失败 1 | 用户支付的Trace ID | AT (Aggregate Trade) | 成功 RL (ReLease) | 失败 RF (RFund)


## API
版本 V1

EndPoint: https://mixswap.exchange/api/v1

### 计算交易路线分布
**/trade/routes**

请求参数：
* payAssetUuid `string`

  支付资产的 UUID
* receiveAssetUuid `string`

  所得资产uuid
* payAmount `string`

  支付的数量



响应：
```
{
    "code": 0,
    "success": true,
    "message": "",
    "data": {
        "id": "72565acb-b827-4890-a109-1dbc4fe62429",
        "payAssetUuid": "c94ac88f-4671-3976-b60a-09064f1811e8",
        "receiveAssetUuid": "31d2ea9c-95eb-3355-b65b-ba096853bc18",
        "payAmount": "1",
        "estimateReceiveAmount": "894.65276378",
        "sources": [
            {
                "exchangeSymbol": "exinswap",
                "routes": [
                    {
                        "routePath": [
                            "c94ac88f-4671-3976-b60a-09064f1811e8",
                            "4d8c508b-91c5-375b-92b0-ee702ed2dac5",
                            "31d2ea9c-95eb-3355-b65b-ba096853bc18"
                        ],
                        "routeIds": [
                            "3",
                            "1",
                            "K"
                        ],
                        "routeAmounts": [
                            "0.1",
                            "89.55985772",
                            "89.48090437"
                        ],
                        "payAmount": "0.1",
                        "receiveAmount": "89.48090437"
                    }
                ]
            },
            {
                "exchangeSymbol": "foxswap",
                "routes": [
                    {
                        "routePath": [
                            "c94ac88f-4671-3976-b60a-09064f1811e8",
                            "31d2ea9c-95eb-3355-b65b-ba096853bc18"
                        ],
                        "routeIds": [
                            "175"
                        ],
                        "routeAmounts": [
                            "0.5",
                            "447.32715486"
                        ],
                        "payAmount": "0.5",
                        "receiveAmount": "447.32715486"
                    },
                    {
                        "routePath": [
                            "c94ac88f-4671-3976-b60a-09064f1811e8",
                            "6cfe566e-4aad-470b-8c9a-2fd35b49c68d",
                            "31d2ea9c-95eb-3355-b65b-ba096853bc18"
                        ],
                        "routeIds": [
                            "22",
                            "176"
                        ],
                        "routeAmounts": [
                            "0.1",
                            "22.60526093",
                            "89.46811033"
                        ],
                        "payAmount": "0.1",
                        "receiveAmount": "89.46811033"
                    },
                    {
                        "routePath": [
                            "c94ac88f-4671-3976-b60a-09064f1811e8",
                            "c6d0c728-2624-429b-8e0d-d9d19b6592fa",
                            "31d2ea9c-95eb-3355-b65b-ba096853bc18"
                        ],
                        "routeIds": [
                            "4",
                            "174"
                        ],
                        "routeAmounts": [
                            "0.3",
                            "0.00482527",
                            "268.37659422"
                        ],
                        "payAmount": "0.3",
                        "receiveAmount": "268.37659422"
                    }
                ]
            }
        ],
        "rank": [
            {
                "symbol": "foxswap",
                "estimateReceiveAmount": "894.48885997"
            },
            {
                "symbol": "exinswap",
                "estimateReceiveAmount": "893.32321988"
            }
        ],
        "bestSourceReceiveAmount": "894.48885997"
    },
    "timestampMs": 1615954360496
}
```


### 查询订单详情
**/order/{TRACE_ID}**

请求参数： 
* TRACE_ID (Path)

  用户发起交易转账时的 Trace ID

响应：
```
{
    "code": 0,
    "success": true,
    "message": "",
    "data": {
        "payAssetUuid": "f5ef6b5d-cc5a-3d90-b2c0-a2fd386e7a3c",
        "payAmount": "0.18950887",
        "receiveAssetUuid": "43d61dcd-e413-450d-80b8-101d5e903357",
        "receiveAmount": "0.00066484",
        "refundAmount": "0.00947551",
        "tradePrice": "270.7920101076950845",
        "orderStatus": "done",
        "refundStatus": "done",
        "minReceiveAmount": "0.00069968",
        "estimateReceiveAmount": "0.00070039",
        "estimateBestSourceReceiveAmount": "0.00069998",
        "profitAmount": "-0.00000014",
        "usdtProfitAmount": "-0.00022717",
        "exchangeOrders": [
            {
                "payAssetUuid": "f5ef6b5d-cc5a-3d90-b2c0-a2fd386e7a3c",
                "payAmount": "0.00947544",
                "receiveAssetUuid": "43d61dcd-e413-450d-80b8-101d5e903357",
                "receiveAmount": "0.00003532",
                "refundAmount": null,
                "minReceiveAmount": "0.00003528",
                "tradeStatus": "done"
            },
            {
                "payAssetUuid": "f5ef6b5d-cc5a-3d90-b2c0-a2fd386e7a3c",
                "payAmount": "0.00947544",
                "receiveAssetUuid": "43d61dcd-e413-450d-80b8-101d5e903357",
                "receiveAmount": null,
                "refundAmount": "0.00947544",
                "minReceiveAmount": "0.00003520",
                "tradeStatus": "done"
            },
            {
                "payAssetUuid": "f5ef6b5d-cc5a-3d90-b2c0-a2fd386e7a3c",
                "payAmount": "0.17055792",
                "receiveAssetUuid": "43d61dcd-e413-450d-80b8-101d5e903357",
                "receiveAmount": "0.00062952",
                "refundAmount": null,
                "minReceiveAmount": "0.00062919",
                "tradeStatus": "done"
            },
            {
                "payAssetUuid": "f5ef6b5d-cc5a-3d90-b2c0-a2fd386e7a3c",
                "payAmount": "0.00000007",
                "receiveAssetUuid": "43d61dcd-e413-450d-80b8-101d5e903357",
                "receiveAmount": null,
                "refundAmount": "0.00000007",
                "minReceiveAmount": "0.00000000",
                "tradeStatus": "done"
            }
        ]
    },
    "timestampMs": 1615954085839
}
```


### 获取支持的资产列表
**/assets**

请求参数： 无

响应：
```
{
    "data": [
        {
            "uuid": "c94ac88f-4671-3976-b60a-09064f1811e8",
            "symbol": "XIN",
            "name": "Mixin",
            "iconUrl": "https://mixin-images.zeromesh.net/UasWtBZO0TZyLTLCFQjvE_UYekjC7eHCuT_9_52ZpzmCC-X-NPioVegng7Hfx0XmIUavZgz5UL-HIgPCBECc-Ws=s128",
            "priceUsdt": "898.14700462",
            "chainAsset": {
                "uuid": "43d61dcd-e413-450d-80b8-101d5e903357",
                "symbol": "ETH",
                "name": "Ether",
                "iconUrl": "https://mixin-images.zeromesh.net/zVDjOxNTQvVsA8h2B4ZVxuHoCF3DJszufYKWpd9duXUSbSapoZadC7_13cnWBqg0EmwmRcKGbJaUpA8wFfpgZA=s128"
            }
        },
        ...
    ],
    "code": 0,
    "success": true,
    "message": "",
    "timestampMs": 1615953867668
}
```


### 获取支持的交易所
**/exchanges**

请求参数： 无

响应：
```
{
    "data": [
        {
            "symbol": "exinswap",
            "name": "ExinSwap",
            "iconUrl": "https://mixswap.exchange/img/exinswap.png"
        },
        {
            "symbol": "foxswap",
            "name": "4swap",
            "iconUrl": "https://mixswap.exchange/img/4swap.png"
        }
    ],
    "code": 0,
    "success": true,
    "message": "",
    "timestampMs": 1615955166862
}
```