# MixSwap Development Documentation

##### [Switch to Chinese version](https://github.com/ExinOne/mixswap-doc/blob/master/README-zh.md)

MixSwap is the MiFi DEX aggregate trading platform under Exin. It is connected to the depth of multiple exchanges at the same time. The goal is to help users exchange more coins and save more money.

## Transaction

To start a transaction, you will need to transfer the coins with specific Memo To MixSwap Bot (Mixin ID:7000103767 UserID:6a4a121d-9673-4a7e-a93e-cb9ca4bb83a2)

### User transfer Memo specification

Each field is separated by | , and encoded in BASE64:
ACTION|FIELD1|FIELD2|FIELD3

| ACTIONS | ACTION | FIELD1 | FIELD2 | FIELD3 |
| ---- | ---- | ---- | ----| ---- |
| aggregate trading | 0 | Target asset UUID | Route ID (optional)| minimum receive amount(optional)|

Among:
* Route ID is a string representing the route distribution of this transaction, which can be obtained through the API ;

  If you do not fill in or fill in 0, MixSwap will automatically calculate the current optimal transaction distribution after receiving the transfer;

   The validity period is 15 minutes.  If the transfer has expired when the transfer is received, the currency will be refunded automatically.

* The minimum receive amount is the minimum receive amount allowed to be traded;

   If the transaction result is less than this amount, the transaction will fail and the coins will be refunded automatically;

   If you leave it blank, the transaction will be initiated at the maximum slippage of 0.2%.  If there is a transaction failure, the currency will be refunded, the transaction distribution will be recalculated, and the transaction will be initiated again, until all the paid coins have been traded.

* **When connecting for the first time, a small test is recommended.**

   **Currently, transfers that do not meet the specifications (cannot be decoded correctly, unused | separated, ACTION is incorrect, etc.) will not be automatically refunded.**

Examples of exchanging XIN with aggregate transaction:

> I have already obtained RouteID `36841c0e-ebd2-41e8-b275-907e04b05419` through the API . I hope to get at least 1.2345 XINs:

Encode `0|c94ac88f-4671-3976-b60a-09064f1811e8|36841c0e-ebd2-41e8-b275-907e04b05419|1.2345` in BASE64:
`MHxjOTRhYzg4Zi00NjcxLTM5NzYtYjYwYS0wOTA2NGYxODExZTh8MzY4NDFjMGUtZWJkMi00MWU4LWIyNzUtOTA3ZTA0YjA1NDE5fDEuMjM0NQ==`

> Hope to continue trading until success:

Encode `0|c94ac88f-4671-3976-b60a-09064f1811e8` in BASE64:
`MHxjOTRhYzg4Zi00NjcxLTM5NzYtYjYwYS0wOTA2NGYxODExZTg=`

### Server transfer Memo specification

 Each field will be separated by `| `and encoded in BASE64:
RESULT|TRACE|SOURCE|TYPE

| field | RESULT | TRACE | SOURCE | TYPE |
| ---- | ---- | ---- | ---- | ---- |
| Parameter verification failed to refund | 1 | Trace ID from transaction | SN (SNapshot) | RF (RFund) |
| Aggregate trading | success 0 / failure 1 | Trace ID from transaction | AT (Aggregate Trade) | Success RL (ReLease) |

## API

Version V1

EndPoint: https://mixswap.exchange/api/v1

### Calculate the distribution of trading routes

**/trade/routes**

Request parameters：

* payAssetUuid `string`

   UUID of the payment asset

* receiveAssetUuid `string`

   Receive asset uuid

* payAmount `string`

  Payment amount



Response ：
```json
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


### Check order details
**/order/{TRACE_ID}**

Request parameters:
* TRACE_ID (Path)

  Trace ID when the user initiates a transaction transfer

Response ：
```json
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


### Get a list of supported assets
**/assets**

Request parameters: None

Response ：
```json
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
    ],
    "code": 0,
    "success": true,
    "message": "",
    "timestampMs": 1615953867668
}
```


### Get supported exchanges
**/exchanges**

Request parameters: None

Response ：
```json
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
