**Table of Contents** 
  - [General API Information](#general-api-information)
  - [HTTP Return Codes](#http-return-codes)
- [Limits](#limits)
  - [General Info on Limits](#general-info-on-limits)
  - [IP Limits](#ip-limits)
- [API Endpoints](#API-endpoints)
  - [Terminology](#terminology)
  - [General endpoints](#general-endpoints)
    - [Test connectivity](#test-connectivity)
    - [Check server time](#check-server-time)
	- [Check user stats](#check-user-stats)
  - [Market Data endpoints](#market-data-endpoints)
    - [Assets](#assets)
    - [Ticker](#ticker)
    - [Trading Indicators](#trading-indicators)
    - [Trading Alarms](#trading-alarms)
    - [Trading Orders](#trading-orders)
    - [Arbitration](#arbitration)
    - [All Assets Last Quote](#all-assets-last-quote)
    - [Volumes](#volumes)


# Public Rest API for Crypto Data HUB (2020-03-24)

## General API Information
* The base endpoint is: **https://api.cryptodatahub.dev**
* All endpoints return either a JSON object or array.
* Data is returned in **descending** order. Newest first,  olders last.
* All time and timestamp related fields are in **microseconds**.

## HTTP Return Codes

* HTTP `401` return code is used when the API Key provided does not exist or its activation is still pending;
* HTTP `404` return code is used for malformed requests;
  the issue is on the sender's side.
* HTTP `403` return code is used when you overcome the daily operations allowed by your API Key. API operations counter is reset once per day at 00:00 UTC Time
* HTTP `429` return code is used when breaking a request rate limit based in the throtling policy (100 req/min or 3 req/sec). API Key will be whitelisted again after 30min.
* HTTP `5XX` return codes are used for internal errors; the issue is on our side.

# LIMITS

## IP Limits
* When a 429 is received, it's your obligation as an API to back off and not spam the API. If you consider your API has been compromised, you can request us to generate a new one while deleting the previous.
* **Repeatedly violating rate limits and/or failing to back off after receiving 429s will result in an automated IP blacklisted.**
* IP bans are tracked continuously and **scale in duration** for repeat offenders, **from 2 minutes to 1 week**.
* **The limits on the API are based on the API keys nor the IPs.**


# Public API Endpoints
## Terminology
* `base asset` refers to the asset that is the `quantity` of a symbol.
* `quote asset` refers to the asset that is the `price` of a symbol.
* `timestamp` refers to the UTC DateTime value in microseconds (YYYY-MM-DD hh:mm:ss.zzzzzz).

## General endpoints
### Test connectivity
```
GET /ping.php
```
Test connectivity to the Rest API.

**Parameters:**
NONE

**Response:**
```javascript
{}
```

### Check server time
```
GET /time.php
```
Test connectivity to the Rest API and get the current server time.

**Parameters:**
NONE

**Response:**
```javascript
{
  "serverTime": "2019-07-08 11:14:15 UTC"
}
```

### Check user stats
```
GET /userstatus.php
```
Get user stats and general info, mainly used to track current API Key status.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
apikey | STRING | YES | 4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7

**Example:**
```
https://api.cryptodahub.dev/userstatus.php?apikey=4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7
```
**Response:**
```javascript
{
  "email": "john.andrews.42@hotmail.com",
  "creation_timestamp": "2019-07-08 11:14:15",
  "apikey": "26263hSgHyysIsjAsKiwhH10Js",
  "apikey_status": "enabled",
  "plan": "premium_1",
  "requests_today": "512",
  "available_requests_today": "4488"
}
```

## Market Data endpoints

### Assets
```
GET /assets.php
```
Current exchange trading rules and symbol information

**Parameters:**
NONE

**Example:**
```
https://api.cryptodahub.dev/assets.php
```
**Response:**
```javascript
{
	"data": [{
			  "marketname": "bitmax",
			  "coinpairname": "eth-btc",		
			  "market_id": "eth-btc",			// ASSET ID IN OUR SYSTEM
			  "market_symbol": "ETHBTC"			// ASSET ID IN EXCHANGE
		  },
		  {
			  "marketname": "kraken",
			  "coinpairname": "xrm-btc",		
			  "market_id": "xrm-btc",			// ASSET ID IN OUR SYSTEM
			  "market_symbol": "XXxrmbtcXX"		// ASSET ID IN EXCHANGE
		  },
		  ...
		  ]
}
```

### Ticker
```
GET /ticker.php
```
Get ticker information for a specific market and asset
**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
apikey | STRING | YES | 4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7
market | STRING | YES | Exchange name (Binance, Kraken, Hitbtc...) Only one market at a time
pair   | STRING | YES | Asset pair name (btc-usdt, eth-btc). Only one asset at a time
from   | STRING | NO | Get data from specific point in time. If Null return values from last hour

**Example:**
```
https://api.cryptodahub.dev/ticker.php?apikey=4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7&market=binance&coinpair=btc-usdt
```
**Response:**
```javascript
{
	"data": [{
				"market": "binance",
				"coinpair": "btc-usdt",
				"ask": "7590.0000000000",
				"bid": "7589.9200000000",
				"last": "7590.0000000000",
				"quote_last": "0.0001317523",
				"volume": "36.4747580000",
				"timestamp": "2020-04-26 10:41:23.692"
			 },
			{
				"market":"binance",
				"coinpair":"btc-usdt",
				"ask":"7595.0000000000",
				"bid":"7594.9900000000",
				"last":"7595.0000000000",
				"quote_last":"0.0001316656",
				"volume":"17.7601530000",
				"timestamp":"2020-04-26	10:42:27.596 "
			},
			...
			}]
}
```


### Trading Indicators
```
GET /indicators.php
```
Returns the required indicator value for the selected exchange and asset
Available indicator values are:
1. sma (Simple Moving Average)
2. ema (Exponential Moving Average)
3. bollinger (Bollinget Bands Up/Down values)
4. maxmin (Maxmin Values in timeframe)
5. rsi (Relative Strenght Index)

Every indicator is presented in different timeframe blocks by default which stands for the aggregated data to retrieve the value (5min,15min,30min,60min,4h,24h)

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
apikey | STRING | YES | 4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7
market | STRING | YES | Exchange name (Binance, Kraken, Hitbtc...) Only one market at a time
pair   | STRING | YES | Asset pair name (btc-usdt, eth-btc). Only one asset at a time
indicator | STRING | YES | Indicator name (sma,ema,bollinger,rsi,maxmin)
from   | STRING | NO | Get data from specific point in time. If Null return values from last hour

**Example:**
```
https://api.cryptodahub.dev/indicators.php?apikey=4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7&market=binance&coinpair=btc-usdt&indicator=ema
```
**Response:**
```javascript
{
	"data": [{
			"market": "binance",
			"coinpair": "btc-usdt",
			"ema5min": "7636.6245166184",
			"ema15min": "7633.0828216897",
			"ema30min": "7621.6355339158",
			"ema60min": "7606.1086782060",
			"ema4h": "7574.4784339860",
			"ema24h": "7554.2763261044",
			"timestamp": "2020-04-26 11: 06: 08.458"
		},
		{
			"market": "binance",
			"coinpair": "btc-usdt",
			"ema5min": "7638.0830110789",
			"ema15min": "7634.1384454644",
			"ema30min": "7622.9710143354",
			"ema60min": "7607.3118272334",
			"ema4h": "7575.0619564949",
			"ema24h": "7554.4040487463",
			"timestamp": "2020-04-26 11: 07: 12.344"
		},
		...
		]
}
```

### Trading Alarms
```
GET /alarms.php
```
Returns the required buy/sell alarm status for the selected exchange and asset based on the provided indicator
Not all the indicators create the same kind of alarms output, so in this case we have 2 alarm categories:
1. Buy/Sell Alarms (Alarms based in indicators like sma, ema...) Values can be buy/sell/neutral
2. Volatility Alarms (Alarms based in data agreggation like Volumes) Values can be volatile/neutral

Available alarm values are:
1. sma (Simple Moving Average)
2. ema (Exponential Moving Average)
3. bollinger (Bollinget Bands Up/Down values)
4. maxmin (Maxmin Values in timeframe)
5. rsi (Relative Strenght Index)


Every alarm is presented in different timeframe blocks by default which stands for the aggregated data to retrieve the value (5min,15min,30min,60min,4h,24h)

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
apikey | STRING | YES | 4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7
market | STRING | YES | Exchange name (Binance, Kraken, Hitbtc...) Only one market at a time
pair   | STRING | YES | Asset pair name (btc-usdt, eth-btc). Only one asset at a time
alarm | STRING | YES | Alarm name (sma,ema,bollinger,rsi,maxmin)
from   | STRING | NO | Get data from specific point in time. If Null return values from last hour

**Example:**
```
https://api.cryptodahub.dev/alarms.php?apikey=4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7&market=binance&coinpair=btc-usdt&alarm=ema
```
**Response:**
```javascript
{
	"data": [{
			"market": "binance",
			"coinpair": "btc-usdt",
			"ema5min": "neutral",
			"ema15min": "neutral",
			"ema30min": "neutral",
			"ema60min": "buy",
			"ema4h": "buy",
			"ema24h": "buy",
			"timestamp": "2020-04-26 11:15:34.470"
		},
		{
			"market": "binance",
			"coinpair": "btc-usdt",
			"ema5min": "sell",
			"ema15min": "neutral",
			"ema30min": "neutral",
			"ema60min": "buy",
			"ema4h": "buy",
			"ema24h": "buy",
			"timestamp": "2020-04-26 11:13:32.413
		},
		...
		]
}
```

### Trading Orders
```
GET /orders.php
```
Returns orderbook data for the selected market and asset based in the bot (trending) value
We are continuosly running bots based in the current intraday market trending and based in different alarm aggregation created from the trading indicators
We have currently 2 bots dessigned to maximize benefits based in 24h market data, every bot is a set of buying/sell conditions based in the market trending

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
apikey | STRING | YES | 4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7
market | STRING | YES | Exchange name (Binance, Kraken, Hitbtc...) Only one market at a time
pair   | STRING | YES | Asset pair name (btc-usdt, eth-btc). Only one asset at a time
botname | STRING | YES | market trending value (bear/bull)
from   | STRING | NO | Get data from specific point in time. If Null return values from last hour

**Example:**
```
https://api.cryptodahub.dev/orders.php?apikey=4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7&market=binance&coinpair=btc-usdt&botname=bull
```
**Response:**
```javascript
{
	"data": [{
			"market": "binance",
			"coinpair": "btc-usdt",
			"botname": "bull",
			"bot_id": "1.1.1",
			"condition": "bear_stop_loss",
			"order": "sell",
			"price": "7635.99",
			"current_coin": "btc",
			"timestamp": "2020-04-27 13:30:50.543"
		},
		{
			"market": "binance",
			"coinpair": "btc-usdt",
			"botname": "bull",
			"bot_id": "1.1.1",
			"condition": "30min_perfection",
			"order": "buy",
			"price": "7714.39",
			"current_coin": "usdt",
			"timestamp": "2020-04-27 08:00:49.584"
		},
		...
		]
}
```

### Exchange Arbitration
```
GET /arbitration.php
```
Returns arbitration data between markets based in the source provided exchange name.
Arbitration difference is presented in percentage data referenced to the main market asset price

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
apikey | STRING | YES | 4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7
market | STRING | YES | Exchange name (Binance, Kraken, Hitbtc...) Only one market at a time
pair   | STRING | YES | Asset pair name (btc-usdt, eth-btc). Only one asset at a time
from   | STRING | NO | Get data from specific point in time. If Null return values from last hour

Exchanges where the required asset is not included will return null value.
Exchange deviation will be always zero for the provided one in parameters

**Example:**
```
https://api.cryptodahub.dev/arbitration.php?apikey=4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7&market=binance&coinpair=btc-usdt
```
**Response:**
```javascript
{
	"data": [{
			"market": "binance",
			"coinpair": "btc-usdt",
			"kraken_gap": null,
			"binance_gap": "0.0000000000",
			"bittrex_gap": null,
			"bitstamp_gap": "0.2111424821",,
			"bitfinex_gap": null,
			"poloniex_gap": "2.2314439923",
			"hitbtc_gap": null,
			"coinbase_gap": "-0.182712221",
			"bitmart_gap": null,
			"bitmax_gap": "-0.0588039281",
			"timestamp": "2020-04-27 17:41:02.770"
		},
		{
			"market": "binance",
			"coinpair": "btc-usdt",
			"kraken_gap": null,
			"binance_gap": "0.0000000000",
			"bittrex_gap": null,
			"bitstamp_gap": "0.1141628891",,
			"bitfinex_gap": null,
			"poloniex_gap": "2.2314439923",
			"hitbtc_gap": null,
			"coinbase_gap": "-0.182712221",
			"bitmart_gap": null,
			"bitmax_gap": "-0.0248039281",
			"timestamp": "2020-04-27 17:40:06.150"
		},
		...
		]
}
```

### All Assets Last Price
```
GET /allassetslast.php
```
Get the last ticker value for all the markets and assets combinations in the system


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
apikey | STRING | YES | 4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7

**Example:**
```
https://api.cryptodahub.dev/allassetslast.php?apikey=4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7
```
**Response:**
```javascript
{
	"data": [{
			"market": "binance",
			"coinpair": "btc-usdt",
			"ask": "7664.6200000000",
			"bid": "7664.6100000000",
			"last": "7664.6200000000",
			"reverse_last": "0.0001304696",
			"volume": "57.7218830000",
			"timestamp": "2020-04-27 17:47:30.358"
		},
		{
			"market": "hitbtc",
			"coinpair": "btc-usd",
			"ask": "7663.5200000000",
			"bid": "7661.1700000000",
			"last": "7662.5200000000",
			"reverse_last": "0.0001304883",
			"volume": "195.6324300000",
			"timestamp": "2020-04-27 17:46:59.131"
		},
		{
			"market": "bitmax",
			"coinpair": "btc-usdt",
			"ask": "7663.5500000000",
			"bid": "7663.3300000000",
			"last": "7661.2300000000",
			"reverse_last": "0.0001305273",
			"volume": "17.3906800002",
			"timestamp": "2020-04-27 17:46:34.907"
		},
		...
		]
}
```

### Volumes
```
GET /volumes.php
```
Get aggregated volume values for base and quote assets
There are different volume categories:
1. volume Will be the ticker volume, or simplified, the amount of volume in negotiation between 2 market tickers cycle
2. volumetoday Will be the volume in negotiation since today's 00:00 UTC
3. volume24h The amount of volume negotiated in the last 24h from the moment you send the query


**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
apikey | STRING | YES | 4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7
market | STRING | YES | Exchange name (Binance, Kraken, Hitbtc...) Only one market at a time
pair   | STRING | YES | Asset pair name (btc-usdt, eth-btc). Only one asset at a time
from   | STRING | NO | Get data from specific point in time. If Null return values from last hour

**Example:**
```
https://api.cryptodahub.dev/volumes.php?apikey=4c93aa4ba3d361d7fd6252a398b6fdec9d3f3ea358b521765d95c1c5112a86f7&market=binance&coinpair=btc-usdt
```

**Response:**
```javascript
{
	"data": [{
			"market": "binance",
			"coinpair": "btc-usdt",
			"volume": "23.0492630000",
			"volumetoday": "53964.6439930000",
			"volume24h": "65678.1916850000",
			"quotevolume": "176709.2492231761",
			"quotevolumetoday": "413724799.9902938600",
			"quotevolume24h": "503527767.5903041400",
			"timestamp": "2020-04-27 17:53:44.990"
		},
		{
			"market": "binance",
			"coinpair": "btc-usdt",
			"volume": "14.3549220000",
			"volumetoday": "53941.5947300000",
			"volume24h": "65670.9656790000",
			"quotevolume": "110044.9756012094",
			"quotevolumetoday": "413516804.6161273000",
			"quotevolume24h": "503434279.6048708000",
			"timestamp": "2020-04-27 17:52:42.536"
		},
		...
		]
}
```

