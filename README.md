# Moonbase Exchange WebSocket API documentation

See also [REST API docs](https://moonbase.exchange/api/v1/swagger/index.html).

## Contents

1. [Authentication](#authentication)  
2. [Rate limits](#rate-limits)  
3. [Channels](#channels)  
4. [Examples](#examples)

## Authentication
API key & secret can be generated in user dashboard.  

### Algorithm
* Build a map (associative array) of ```key``` and ```timestamp```, where:  
```key``` is your API key and ```timestamp``` is current UNIX timestamp (number of seconds since 1 January 1970).  
* Encode the map into query string (e.g. ```“key=XXX&timestamp=YYY”```).
* Build request string just as if you were signing a request to ```GET /users/verify``` (e.g. ```“GET\n/users/verify\n{{query_string}}”```).
* Calculate auth_signature via HMAC256 algorithm using API secret and request string.
* Add a ```signature``` parameter to you ```key``` and ```timestamp``` request parameters.
* Send request.

## Rate limits
* Subscription: max 20 per hour for a single key  
* Requests: max 300 per minute for a single key  
* Exclusions:  
    * Cancelling an order has no limits
    
If you exceed the limit, you will see a message like:  
```
{
    "status": "error",
    "type": "rate_limit_exceeded",
    "details": {
        "retry_after": 30
    }
}
```

## Channels
All messages have the same fields:  
1. **ref** - unique message identifier, usually is an incremented value  
2. **join_ref** - identifier (**ref**) of the join message to actual channel (thus for join messages **ref** and **join_ref** will always be the same)  
3. **topic** - topic name, e.g. **/tickers** or **/orders/ETH:BTC**  
4. **event** - event name, e.g. **place** for orders channel or **get_snapshot** for candles channel  
5. **payload** - message payload  

**ref** and **join_ref** is used to track responses. For replies, the **ref** matches the original message

* [Tickers](#tickers)
* [Trades](#trades)
* [Candles](#candles)
* [Orderbooks](#orderbooks)
* [Orders](#orders)
* [Balances](#balances)
* [Positions](#positions)
* [Deposits](#deposits)
* [Withdrawals](#withdrawals)

### Tickers
#### Join
**Topic:** ```/tickers```  

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|symbols|[string]|✖|[]|

**Example *out* frame:**
```
{
    "ref": "1",
    "join_ref": "1",
    "topic": "/tickers",
    "event": "phx_join",
    "payload": {
        "symbols": ["ETC_D:BTC_D"]
    }
}
```

**Example reply frame:**
```
{
    "ref": "1",
    "join_ref": "1",
    "topic": "/tickers",
    "event": "phx_reply",
    "payload": {
        "status":"ok",
        "response":[
            {
                "symbol": "ETC_D:BTC_D",
                "quote_amount": 3.82,
                "mark_price": 1.0,
                "bid_price": 1.0,
                "base_amount": 3.82,
                "ask_price": null
            }
        ]
    }
}
```

#### Upsert
**Example *in* frame:**
```
{
    "ref": null,
    "join_ref": null
    "topic": "/tickers",
    "event": "upsert",
    "payload": {
        "data": [
            {
                "updated_at": "2019-02-17T20:37:41.689411",
                "symbol": "ETC_D:BTC_D",
                "quote_amount": 1.0,
                "mark_price": 1.0,
                "bid_price": 1.0,
                "base_amount": 1.0,
                "ask_price": null
            }
        ]
    }
}
```

### Trades
#### Join

**Topic:** ```/trades/<symbol>```  

**Example *out* frame:**
```
{
    "ref": "2",
    "join_ref": "2",
    "topic": "/trades/ETC_D:BTC_D",
    "event": "phx_join",
    "payload": {}
}
```

**Example reply frame:**
```
{
    "ref": "2",
    "join_ref": "2",
    "topic": "/trades/ETC_D:BTC_D",
    "event": "phx_reply",
    "payload": {
        "status":"ok",
        "response":[
            {
                "timestamp": "2019-02-14T22:46:56.639608",
                "symbol": "ETC_D:BTC_D",
                "price": 1.0,
                "id": 10,
                "amount": 1.0
            },
            {
                "timestamp": "2019-02-14T22:45:51.296859",
                "symbol": "ETC_D:BTC_D",
                "price": 1.0,
                "id": 5,
                "amount": 0.82
            }            
        ]
    }
}
```

#### Upsert
**Example *in* frame:**
```
{
    "ref": null,
    "join_ref": null,
    "topic": "/trades/ETC_D:BTC_D",
    "event": "insert",
    "payload": {
        "data": [
            {
                "timestamp": "2019-02-17T20:37:41.682639",
                "taker_order_id": 10,
                "symbol": "ETC_D:BTC_D",
                "price": 1,
                "maker_order_id": 10,
                "id": 3,
                "amount": 1
            }
        ]
    }
}
```

### Candles
#### Join

**Topic:** ```/candles/<symbol>```  

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|from|integer|✓|-|
|to|integer|✓|-|
|resolution|integer|✓|-|


**Example *out* frame:**
```
{
    "ref": "3",
    "join_ref": "3",
    "topic": "/candles/ETC_D:BTC_D",
    "event": "phx_join",
    "payload": {
        "from": 1550174340,
        "to": 1550184359,
        "resolution": 60
    }
}
```

**Example reply frame:**
```
{
    "ref": "3",
    "join_ref": "3",
    "topic": "/candles/ETC_D:BTC_D",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": {
            "data": [
                [
                    {
                        "volume": 1.0,
                        "timestamp": 1550184300,
                        "symbol": "ETC_D:BTC_D",
                        "sells": 0.0,
                        "resolution": 60,
                        "open": 1.0,
                        "low": 1.0,
                        "id": 69,
                        "high": 1.0,
                        "close": 1.0,
                        "buys": 1.0
                    }
                ],
                1550089080 //nearest candle from the past
            ]
        }
    }
}
```

#### Upsert
**Example *in* frame:**
```
{
    "ref": null,
    "join_ref": null,
    "topic": "/candles/ETC_D:BTC_D",
    "event": "upsert",
    "payload": {
        "data": [
            {
                "volume": 1.0,
                "timestamp": 1550435820,
                "symbol": "ETC_D:BTC_D",
                "sells": 0.0,
                "resolution": 60,
                "open": 1.0,
                "low": 1.0,
                "id": 33,
                "high": 1.0,
                "close": 1.0,
                "buys": 1.0
            }
        ]
    }
}
```

#### Get snapshot
Same as the join, just use the ```get_snapshot``` event instead of ```phx_join```.  

### Orderbooks
#### Join
**Topic:** ```/orderbooks/<symbol>```  

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|limit|integer|✖|null|

**Example *out* frame:**
```
{
    "ref": "4",
    "join_ref": "4",
    "topic": "/orderbooks/ETC_D:BTC_D",
    "event": "phx_join",
    "payload": {}
}
```

**Example reply frame:**
```
{
    "ref": "4",
    "join_ref": "4",
    "topic": "/orderbooks/ETC_D:BTC_D",
    "event": "phx_reply",
    "payload": {
        "status":"ok",
        "response":[
            [
                0.5, //price
                1.0  //amount (negative amount is for short orders)
            ],
            [
                1.0,
                -1.0
            ]
        ]
    }
}
```

#### Upsert
**Example *in* frame:**
```
{
    "ref": null,
    "join_ref": null,
    "topic": "/orderbooks/ETC_D:BTC_D",
    "event": "upsert",
    "payload": {
        "data": [
            [
                1.0,
                1.0
            ]
        ]
    }
}
```

### Orders
#### Join
**Topic:** ```/orders/<symbol>```  

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|status|string|✖|null|
|limit|integer|✖|1000|

**Example *out* frame:**
```
{
    "ref": "5",
    "join_ref": "5",
    "topic": "/orders/ETC_D:BTC_D",
    "event": "phx_join",
    "payload": {
        "status": "open",
        "limit": 100
    }
}
```

**Example reply frame:**
```
{
    "ref": "5",
    "join_ref": "5",
    "topic": "/orders/ETC_D:BTC_D",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": [
            {
                "user_id": 5,
                "updated_at": "2019-02-17T16:40:23.953166",
                "triggered_at": "2019-02-17T16:40:23.951320",
                "trigger_price": null,
                "symbol": "ETC_D:BTC_D",
                "requested_amount": 1.0,
                "limit_price": 0.5,
                "is_liquidation": false,
                "is_active": true,
                "inserted_at": "2019-02-17T16:40:23.953157",
                "id": 30,
                "filled_total": 0.0,
                "filled_amount": 0.0,
                "fee": 0.0,
                "external_id": null,
                "cancelled_at": null
            },
            {
                "user_id": 5,
                "updated_at": "2019-02-17T16:40:12.873726",
                "triggered_at": "2019-02-17T16:40:12.871981",
                "trigger_price": null,
                "symbol": "ETC_D:BTC_D",
                "requested_amount": -1.0,
                "limit_price": 1.0,
                "is_liquidation": false,
                "is_active": true,
                "inserted_at": "2019-02-17T16:40:12.873716",
                "id": 29,
                "filled_total": 0.0,
                "filled_amount": 0.0,
                "fee": 0.0,
                "external_id": null,
                "cancelled_at": null
            }
        ]
    }
}
```

#### Place
**Topic:** ```/orders/<symbol>```  

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|limit_price|float|✖|null|
|trigger_price|float|✖|null|
|requested_amount|float|✓|-|

**Example *out* frame:**
```
{
    "ref": "6",
    "join_ref": "5",
    "topic": "/orders/ETC_D:BTC_D",
    "event": "place",
    "payload": {
        "limit_price": 1.0,
        "requested_amount": 1.0
    }
}
```

**Example reply frame:**
```
{
    "ref": "6",
    "join_ref": "5",
    "topic": "/orders/ETC_D:BTC_D",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": {
            "updated_at": "2019-02-17T20:37:41.679118",
            "triggered_at": "2019-02-17T20:37:41.678723",
            "trigger_price": null,
            "symbol": "ETC_D:BTC_D",
            "requested_amount": 1.0,
            "limit_price": 1.0,
            "is_liquidation": false,
            "is_active": true,
            "inserted_at": "2019-02-17T20:37:41.679112",
            "id": 10,
            "filled_total": 0.0,
            "filled_amount": 0.0,
            "fee": 0.0,
            "external_id": null,
            "cancelled_at": null
        }
    }
}
```

#### Cancel
**Topic:** ```/orders/<symbol>```  

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|id|integer|✓|-|

**Example *out* frame:**
```
{
    "ref": "7",
    "join_ref": "5",
    "topic": "/orders/ETC_D:BTC_D",
    "event": "cancel",
    "payload": {
        "id": 10
    }
}
```

**Example reply frame:**
```
{
    "ref": "7",
    "join_ref": "5",
    "topic": "/orders/ETC_D:BTC_D",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": {
            "updated_at": "2019-02-17T21:07:50.745530",
            "triggered_at": "2019-02-17T20:37:41.678723",
            "trigger_price": null,
            "symbol": "ETC_D:BTC_D",
            "requested_amount": 1,
            "limit_price": 1,
            "is_liquidation": false,
            "is_active": false,
            "inserted_at": "2019-02-17T20:37:41.679112",
            "id": 10,
            "filled_total": 0,
            "filled_amount": 0,
            "fee": 0,
            "external_id": null,
            "cancelled_at": "2019-02-17T21:07:50.745379"
        }
    }
}
```

#### Upsert
**Example *in* frame:**
```
{
    "ref": null,
    "join_ref": null,
    "topic": "/orders/ETC_D:BTC_D",
    "event": "upsert",
    "payload": {
        "data": [
            {
                "updated_at": "2019-02-17T21:07:50.745530",
                "triggered_at": "2019-02-17T20:37:41.678723",
                "trigger_price": null,
                "symbol": "ETC_D:BTC_D",
                "requested_amount": 1,
                "limit_price": 1,
                "is_liquidation": false,
                "is_active": false,
                "inserted_at": "2019-02-17T20:37:41.679112",
                "id": 10,
                "filled_total": 0,
                "filled_amount": 0,
                "fee": 0,
                "external_id": null,
                "cancelled_at": "2019-02-17T21:07:50.745379"
            }
        ]
    }
}
```

### Balances
#### Join
**Topic:** ```/balances```  

**Example *out* frame:**
```
{
    "ref": "8",
    "join_ref": "8",
    "topic": "/balances",
    "event": "phx_join",
    "payload": {}
}
```

**Example reply frame:**
```
{
    "ref": "8",
    "join_ref": "8",
    "topic": "/balances",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": [
            {
                "withdrawn_amount": 0,
                "pledged_amount": 0,
                "placed_amount": 0,
                "offered_amount": 0,
                "loaned_amount": 0,
                "id": 15,
                "drawn_amount": 0,
                "borrowed_amount": 0,
                "available_amount": 100,
                "assigned_amount": 0,
                "asset": "BTC_D"
            }
        ]
    }
}
```

#### Upsert
**Example *in* frame:**
```
{
    "ref": null,
    "join_ref": null,
    "topic": "/balances",
    "event": "upsert",
    "payload": {
        "data": [
            {
              "withdrawn_amount": 0,
              "pledged_amount": 0,
              "placed_amount": 0,
              "offered_amount": 0,
              "loaned_amount": 0,
              "id": 15,
              "drawn_amount": 0,
              "borrowed_amount": 0,
              "available_amount": 90,
              "assigned_amount": 10,
              "asset": "BTC_D"
            }
        ]
    }
}
```


### Positions
#### Join
**Topic:** ```/positions```  

**Example *out* frame:**
```
{
    "ref": "9",
    "join_ref": "9",
    "topic": "/positions",
    "event": "phx_join",
    "payload": {}
}
```

**Example reply frame:**
```
{
    "ref": "9",
    "join_ref": "9",
    "topic": "/positions",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": [
            {
                "user_id": 5,
                "total": -1,
                "symbol": "ETC_D:BTC_D",
                "price": 1,
                "margin": 10,
                "id": 1,
                "fee": -0.001,
                "amount": 1
            },
            {
                "user_id": 5,
                "total": 0,
                "symbol": "ICO_D:BTC_D",
                "price": 0,
                "margin": 10,
                "id": 2,
                "fee": 0,
                "amount": 0
            }
        ]
    }
}
```

#### Assign
**Topic:** ```/positions``` 

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|symbol|string|✓|-|
|margin|decimal|✓|-|

**Example *out* frame:**
```
{
    "ref": "10",
    "join_ref": "9",
    "topic": "/positions",
    "event": "assign",
    "payload": {
        "ETC_D:BTC_D": "10",
        "ICO_D:BTC_D": "10"
    }
}
```

**Example reply frame:**
```
{
    "ref": "10",
    "join_ref": "9",
    "topic": "/positions",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": {
            "data": [
                {
                    "total": 0,
                    "symbol": "ICO_D:BTC_D",
                    "price": 0,
                    "margin": 10,
                    "id": 2,
                    "fee": 0,
                    "amount": 0
                },
                {
                    "total": 0,
                    "symbol": "ETC_D:BTC_D",
                    "price": 0,
                    "margin": 10,
                    "id": 1,
                    "fee": 0,
                    "amount": 0
                }
            ]
        }
    }
}
```

#### Upsert
**Example *in* frame:**
```
{
    "ref": null,
    "join_ref": null,
    "topic": "/positions",
    "event": "upsert",
    "payload": {
        "data": [
            {
                "total": 0.0,
                "symbol": "ETC_D:BTC_D",
                "price": 0.0,
                "margin": 10.0,
                "id": 1,
                "fee": 0.0,
                "amount": 0.0
            }
        ]
    }
}
```

### Deposits
#### Join
**Topic:** ```/deposits``` 

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|asset|string|✖|-|
|limit|integer|✖|1000|

**Example *out* frame:**
```
{
    "ref": "11",
    "join_ref": "11",
    "topic": "/deposits",
    "event": "phx_join",
    "payload": {}
}
```

**Example reply frame:**
```
{
    "ref": "11",
    "join_ref": "11",
    "topic": "/deposits",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": [
            {
                "updated_at": "2019-02-17T20:36:30.299646",
                "transaction_hash": "BONUS-48ac4ca8-1ecf-44d3-8ded-7e9ba13176ed",
                "is_processed": true,
                "inserted_at": "2019-02-17T20:36:30.299632",
                "id": 15,
                "confirmation_count": 3,
                "asset": "BTC_D",
                "amount": 100,
                "address": "0a421c84-149c-42bd-8f14-887ed660964b"
            }
        ]
    }
}
```

#### Get deposit address
**Topic:** ```/deposits``` 

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|asset|string|✓|-|

**Example *out* frame:**
```
{
    "ref": "12",
    "join_ref": "11",
    "topic": "/deposits",
    "event": "get_deposit_address",
    "payload": {
        "asset": "BTC_T"
    }
}
```

**Example reply frame:**
```
{
    "ref": "12",
    "join_ref": "11",
    "topic": "/deposits",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": {
            "asset": "BTC_T",
            "address": "some address"
        }
    }
}
```

### Withdrawals
#### Join
**Topic:** ```/withdrawals``` 

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|asset|string|✖|-|
|limit|integer|✖|1000|

**Example *out* frame:**
```
{
    "ref": "13",
    "join_ref": "13",
    "topic": "/withdrawals",
    "event": "phx_join",
    "payload": {}
}
```

**Example reply frame:**
```
{
    "ref": "13",
    "join_ref": "13",
    "topic": "/withdrawals",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": [
            {
                "inserted_at": "2017-12-24T12:30:01.234Z",
                "id": 123,
                "asset": "ETH",
                "amount": 10.25,
                "address": "3FfmbHfnpaZjKFvyi1okTjJJusN455paPH"
            }
        ]
    }
}
```

#### Create
**Topic:** ```/withdrawals``` 

**Parameters:**  

|Name|Type|Required|Default|  
|:---:|:---:|:---:|:---:|
|asset|string|✓|-|
|address|string|✓|-|
|amount|float|✓|-|

**Example *out* frame:**
```
{
    "ref": "14",
    "join_ref": "13",
    "topic": "/withdrawals",
    "event": "create_withdrawal",
    "payload": {
        "amount": "1.0",
        "address": "some address",
        "asset": "BTC_T"
    }
}
```

**Example reply frame:**
```
{
    "ref": "14",
    "join_ref": "13",
    "topic": "/withdrawals",
    "event": "phx_reply",
    "payload": {
        "status": "ok",
        "response": {
            "inserted_at": "2017-12-24T12:30:01.234Z",
            "id": 123,
            "asset": "ETH",
            "amount": 10.25,
            "address": "3FfmbHfnpaZjKFvyi1okTjJJusN455paPH"
        }
    }
}
```
