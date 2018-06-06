## [CoinMarketCap](https://coinmarketcap.com/zh/api/#endpoint_ticker)接口分析

### 币列表(<https://api.coinmarketcap.com/v2/listings/>)

- 字段

```latex
"name" : 名字
"symbol" ： 符号
"website_slug" ： 网站简称
"timestamp" : 时间戳
"num_cryptocurrencies" : 数字货币数量
```

```json
{
    "data": [
        {
            "id": 1, 
            "name": "Bitcoin", 
            "symbol": "BTC", 
            "website_slug": "bitcoin"
        }, 
        {
            "id": 2, 
            "name": "Litecoin", 
            "symbol": "LTC", 
            "website_slug": "litecoin"
        }, 
        ...
    },
    "metadata": {
        "timestamp": 1525137187, 
        "num_cryptocurrencies": 1602, 
        "error": null
    }
]            
```



## 笔记

- rank 市值排名
- circulating_supply ： 流通总量
- total_supply：流通总量
- max_supply 发行总量
- quotes ： 报价
- percent_change_1h : 一小时涨跌幅
- percent_change_24h ：二小时涨跌幅
- percent_change_7d：七天涨跌幅
- volume_24：24小时成交额
- market_cap：流通总市值
- bitcoin_percentage_of_market_cap : 比特币占比
- total_market_cap : 市场总量
- total_volume_24h : 24小时交易总量

### 

### 币市值，当前价格，24小时涨跌幅，1小时涨跌幅，七天涨跌幅，24小时成交额，流通总市值<https://api.coinmarketcap.com/v2/ticker/> 



```json
{
    "data": {
        "1": {
            "id": 1, 
            "name": "Bitcoin", 
            "symbol": "BTC", 
            "website_slug": "bitcoin", 
            "rank": 1, 
            "circulating_supply": 17008162.0, 
            "total_supply": 17008162.0, 
            "max_supply": 21000000.0, 
            "quotes": {
                "USD": {
                    "price": 9024.09, 
                    "volume_24h": 8765400000.0, 
                    "market_cap": 153483184623.0, 
                    "percent_change_1h": -2.31, 
                    "percent_change_24h": -4.18, 
                    "percent_change_7d": -0.47
                }
            }, 
            "last_updated": 1525137271
        }, 
        "1027": {
            "id": 1027, 
            "name": "Ethereum", 
            "symbol": "ETH", 
            "website_slug": "ethereum", 
            "rank": 2, 
            "circulating_supply": 99151888.0, 
            "total_supply": 99151888.0, 
            "max_supply": null, 
            "quotes": {
                "USD": {
                    "price": 642.399, 
                    "volume_24h": 2871290000.0, 
                    "market_cap": 63695073558.0, 
                    "percent_change_1h": -3.75, 
                    "percent_change_24h": -7.01, 
                    "percent_change_7d": -2.32
                }
            }, 
            "last_updated": 1525137260
        } 
        ...
    },
    "metadata": {
        "timestamp": 1525137187, 
        "num_cryptocurrencies": 1602, 
        "error": null
    }
]                          
```

### 币币互转(<https://api.coinmarketcap.com/v2/ticker/?convert=BTC&limit=10> )

```json
{
    "data": {
        "1": {
            "id": 1, 
            "name": "Bitcoin", 
            "symbol": "BTC", 
            "website_slug": "bitcoin", 
            "rank": 1, 
            "circulating_supply": 17078075.0, 
            "total_supply": 17078075.0, 
            "max_supply": 21000000.0, 
            "quotes": {
                "USD": {
                    "price": 7635.04, 
                    "volume_24h": 4712820000.0, 
                    "market_cap": 130391785748.0, 
                    "percent_change_1h": 0.44, 
                    "percent_change_24h": 2.71, 
                    "percent_change_7d": 1.6
                }, 
                "CNY": {
                    "price": 48840.4999808093, 
                    "volume_24h": 30147384312.27051, 
                    "market_cap": 834101721710.0, 
                    "percent_change_1h": 0.44, 
                    "percent_change_24h": 2.71, 
                    "percent_change_7d": 1.6
                }
            }, 
            "last_updated": 1528266575
        }
```

### 具体货币(<https://api.coinmarketcap.com/v2/ticker/1/> )

```json
{
    "data": {
        "id": 1, 
        "name": "Bitcoin", 
        "symbol": "BTC", 
        "website_slug": "bitcoin", 
        "rank": 1, 
        "circulating_supply": 17008162.0, 
        "total_supply": 17008162.0, 
        "max_supply": 21000000.0, 
        "quotes": {
            "USD": {
                "price": 9024.09, 
                "volume_24h": 8765400000.0, 
                "market_cap": 153483184623.0, 
                "percent_change_1h": -2.31, 
                "percent_change_24h": -4.18, 
                "percent_change_7d": -0.47
            }
        }, 
        "last_updated": 1525137271
    },
    "metadata": {
        "timestamp": 1525237332, 
        "error": null
    }

}               
```

### 具体货币互转(https://api.coinmarketcap.com/v2/ticker/1/?convert=CNY)

```JSON
{
    "data": {
        "id": 1, 
        "name": "Bitcoin", 
        "symbol": "BTC", 
        "website_slug": "bitcoin", 
        "rank": 1, 
        "circulating_supply": 17078075.0, 
        "total_supply": 17078075.0, 
        "max_supply": 21000000.0, 
        "quotes": {
            "USD": {
                "price": 7637.54, 
                "volume_24h": 4717060000.0, 
                "market_cap": 130434480936.0, 
                "percent_change_1h": 0.44, 
                "percent_change_24h": 2.74, 
                "percent_change_7d": 1.63
            }, 
            "CNY": {
                "price": 48856.4922021929, 
                "volume_24h": 30174507119.73696, 
                "market_cap": 834374838066.0, 
                "percent_change_1h": 0.44, 
                "percent_change_24h": 2.74, 
                "percent_change_7d": 1.63
            }
        }, 
        "last_updated": 1528266875
    }, 
    "metadata": {
        "timestamp": 1528266628, 
        "error": null
    }
}
```



## 流通总量(https://api.coinmarketcap.com/v2/global/?convert=CNY)

```json
{
    "data": {
        "active_cryptocurrencies": 1644, 
        "active_markets": 11216, 
        "bitcoin_percentage_of_market_cap": 37.77, 
        "quotes": {
            "USD": {
                "total_market_cap": 345376710236.0, 
                "total_volume_24h": 15338509256.0
            }, 
            "CNY": {
                "total_market_cap": 2209336324320.0, 
                "total_volume_24h": 98118734289.0
            }
        }, 
        "last_updated": 1528266875
    }, 
    "metadata": {
        "timestamp": 1528266861, 
        "error": null
    }
}
```

