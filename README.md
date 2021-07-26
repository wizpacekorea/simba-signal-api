# LOT-Simba Signal Maker's API

API Version | ``2``
---- | ----
Endpoint | ``https://simba.leagueoftraders.io/v1/<endpoint>``
Content-Type | ``application/json``

# Constants

#### Headers (caps don't matter)
Header | Meaning
---- | ----
Access-Token | ``Signal maker's access token``

# Contents
## Bot Maker
* [Bot Maker Profile](#bot-maker---profile)

## Signal Source
* [Create source](#source---create)
* [Get all](#source---get-all)

## Signal
* [New signal](#signal---new-signal)


# Bot Maker - Profile
## Request
Description | Method | Path
---- | ---- | ----
Get my profile | ``GET`` | ``/bot_maker/profile``

### Response
```json
    {
      "status": 0,
      "data": {
        "id":"5423cbd2234432",
        "name":"SiON",
        "logo":"https://image.com/sample.jpg",
        "phone":"+821000001111",
        "email":"support@sion.com"
      }
```

# Source - Create
## Request
Description | Method | Path
---- | ---- | ----
Create new signal source | ``POST`` | ``/signal_source/create``

```
For Spot symbols use format BTC/KRW, BTC/USDT.....

For futures, exchange_fill if True we'll automatically match the symbols and add if they exist in all our supported exchanges. E.g if you send BTCUSDT Binance, we'll match with BTC-PERP on FTX and XBTUSD on Bitmex and support all exchanges automatically.
```

### Body
```json
  {
        "identifier":"PKG000000000001",
        "name":"Binance 10X Algorithm",
        "exchange":"binance",
        "exchanges":["binance","ftx"],
        "exchange_fill":true,
        "logo":"https://image.com/sample.jpg",
        "short_description":{
          "en":"description in english",
          "ko":"한국어로"
        },
        "long_description":{
          "en":"long description in english",
          "ko":"한국어로"
        },
        "symbols":["BTCUSDT"]
  }
```

### Response
```json
  {
    "status":0,
    "data":{
      "identifier":"PKG000000000001",
      "name":"Bitmex 10X Algorithm",
      "exchange":"bitmex",
      "symbols":["BTCUSDT"],
      "logo":"https://image.com/sample.jpg",
      "short_description":{
          "en":"description in english",
          "ko":"한국어로"
       },
      "long_description":{
          "en":"long description in english",
          "ko":"한국어로"
       },
      "status":"active"
    }
```

# Source - Get all
## Request
Description | Method | Path
---- | ---- | ----
Get all my signal sources | ``GET`` | ``/signal_source/all``

### Response
```json
  {
    "status": 0,
    "data": [
        {
            "bot_maker_id": "5ff6c4fa4ebfb905ac7f7bce",
            "identifier": "PKG000000000001",
            "name": "Bitmex 10X Algorithm",
            "exchange": "bitmex",
            "symbols":["BTCUSDT"],
            "status": "active",
           "short_description":{
                "en":"description in english",
                "ko":"한국어로"
             },
            "long_description":{
                "en":"long description in english",
                "ko":"한국어로"
             },
            "created_at": "2021-01-07T10:25:03.917Z"
        }
    ]
}
```


# Signal - New signal

## Request
Description | Method | Path
---- | ---- | ----
Execute new signal | ``POST`` | ``/signal/make``

### Body (test on api test link above)
```json
{
  "unique_id": "76cfa72f-275b-4537-970a-c495b20a49c6",
  "source_identifier":"PKG000000001",
  "symbol": "XBTUSD",
  "price": "12000",
  "wallet_balance":"1000.50",
  "wallet_balance_symbol":"USDT",
  "current_position_quantity":"120",
  "order_quantity":"100",
  "side": "sell",
  "type": "limit",
  "timestamp": 1607402327,
  "valid_before": 1607402327
}
```

## Response
```json
{
  "status":0
}
```

### unique_id
* Required all the time
* Generated by the signal maker, and should be unique, if two signals with same id are sent the last one will be ignored and an error returned. 
* Using uuid is recommended.

### source_identifier
* Required all the time
* The identifier of the source (from "signal_source/create")

### symbol
* Required all the time
* Should be the same as the symbols provided by the exchange originally through the API.
* For future's API e.g. Bitmex (XBTUSD, ETHUSD.....) 
* For spot exchanges it can be different depending on the market e.g. Binance (BTC/USDT).
* Test the respective exchange's API just to be sure. 

### price
* Required only when making a limit order. 
* Provide in string format for accuracy (json can have issues with scientific notion)
* We'll automatically truncate the decimals based on exchange's symbol precision. E.g if on Bitmex XBTUSD is 2 precision 1500.9954 will be truncated to 1500.99.
* In string format for accuracy

### wallet_balance
* Required all the time
* Must go with wallet_balance_symbol, if it's spot say BTC/USDT then USDT is the symbol, wallet_balance is the amount of USDT in the account. For future's e.g on Bitmex the balance is XBT. Then use XBT
* Float string for accuracy

### wallet_balance_symbol
* Required all the time
* The balance of the base currency used to trade. In spot e.g ETH/USDT the wallet_balance_symbol is USDT.
* Make sure it matches the symbol provided in the exchange. 

### current_position_quantity
* Required all the time, can be -, 0, + 
* It's the quantity of the position that's already open. E.g I have a short position open of 500 XBTUSD on Bitmex then ``current_position_quantity = -500``
* Negative if you have a short position
* 0 if nothing means this is the first order to open the position
* Positive if you have a long position

### order_quantity
* Required all the time, always + even for sell orders because we have "side" you can put sell there
* The amount you are making for this particular order. 
* Example buying or selling 200 contracts of XBTUSD on Bitmex then ``order_quanttity = 200``

### side
* Required all the time
* buy
* sell
* close
* switchSellToBuy
* switchBuyToSell

### type
* Required all the time
* limit
* market

### timestamp
* Required all the time
* The time the signal is sent, if you are resending in case of network errors don't change, use the first timestamp from the first time you wished to send the order.
* Unix up to seconds only format e.g. 1607402327


### valid_before
* Required all the time
* The deadline for executing this order
* Unix up to seconds only format e.g. 1607402327


# Match Position
## Request
Description | Method | Path
---- | ---- | ----
Execute a match order | ``POST`` | ``/match``

### Body (test on api test link above)
```json
{
  "wallet_balance":"0.5",
  "wallet_balance_symbol":"XBT",
  "timestamp":1607402327,
  "open_positions":[
    {
      "symbol":"XBTUSD",
      "amount":"2500",
      "avg_entry_price":"1950.5"
     },
     {
      "symbol":"ETHUSD",
      "amount":"300",
      "avg_entry_price":"450.5"
     }
  ]
}
```

## Response
```json
{
  "status":0
}
```

### wallet_balance
* Required all the time
* Must go with wallet_balance_symbol, if it's spot say BTC/USDT then USDT is the symbol, wallet_balance is the amount of USDT in the account. For future's e.g on Bitmex the balance is XBT. Then use XBT
* Float string for accuracy

### wallet_balance_symbol
* Required all the time
* The balance of the base currency used to trade. In spot e.g ETH/USDT the wallet_balance_symbol is USDT.
* Make sure it matches the symbol provided in the exchange. 

### timestamp
* Required all the time
* The time the signal is sent, if you are resending in case of network errors don't change, use the first timestamp from the first time you wished to send the order.
* Unix up to seconds only format e.g. 1607402327

### open_positions
* Required all the time
* All positions that are currently being held by the user
* Return an empty array if no position


### open_position > symbol
* Required all the time
* The symbol, matching the exchange symbol of the position being held.

### open_position > amount
* Required all the time
* Positive for long, Negative for short.
* Float string for accuracy

### open_position > avg_entry_price
* Required all the time
* The average price you opened the position at.
* Float string for accuracy


## Note
* For /match, send the snapshot every 5-10 seconds if you can. The quicker we match your position list the more close the data
* For bots that only issue buy or sell signal. Send only:
```
  {
    "unique_id": "76cfa72f-275b-4537-970a-c495b20a49c6",
    "source_identifier":"PKG000000001",
    "price": "12000",
    "side": "sell",
    "type": "limit",
    "timestamp": 1607402327,
    "valid_before": 1607402327
  }
```
* - price not required if it's just market order. 


## Error handling
* We always return http status 200 in all cases including when there's an error on your end. Any other status means the request didn't reach our server. If you don't receive 200 in case our servers are offline, try resending the signal every 5 seconds until the signal reaches the server.
* Read the status field in the response to see the status code.

## Status
* 0 = No error
* 4000 = Server error
* 4001 = Unauthorized (check your access token)
* 4002 = Invalid request (make sure you provide all the required parameters)
* 4004 = Signal duplicate (when you send the signal with same id twice)
* 4005 = Wrong request

