# cctx Python使用

- 安装

```shell
pip install ccxt
```

- 导入使用

```python
import ccxt
print(ccxt.exchanges) # print a list of all available exchange classes
```

## 用户手册

### 概要

cctx库收集了各种各样的交易所，没有交易所都有其对应的公开的和私有的API,cctx对交易所进行了封装，所有交易所都继承自基础交易所。并且分享了一些公共的方法。为了访问各种各种的交易所，当你使用一个交易所的时候，需要实例化相对应的交易所。支持的交易所经常更新，并定期添加新的交易所。 

项目结构基本如下

```
                                 User
    +-------------------------------------------------------------+
    |                            CCXT                             |
    +------------------------------+------------------------------+
    |            Public            |           Private            |
    +=============================================================+
    │                              .                              |
    │                    The Unified CCXT API                     |
    │                              .                              |
    |       loadMarkets            .           fetchBalance       |
    |       fetchMarkets           .            createOrder       |
    |       fetchCurrencies        .            cancelOrder       |
    |       fetchTicker            .             fetchOrder       |
    |       fetchTickers           .            fetchOrders       |
    |       fetchOrderBook         .        fetchOpenOrders       |
    |       fetchOHLCV             .      fetchClosedOrders       |
    |       fetchTrades            .          fetchMyTrades       |
    |                              .                deposit       |
    |                              .               withdraw       |
    │                              .                              |
    +=============================================================+
    │                              .                              |
    |                     Custom Exchange API                     |
    |                      (Derived Classes)                      |
    │                              .                              |
    |       publicGet...           .          privateGet...       |
    |       publicPost...          .         privatePost...       |
    |                              .          privatePut...       |
    |                              .       privateDelete...       |
    |                              .                   sign       |
    │                              .                              |
    +=============================================================+
    │                              .                              |
    |                      Base Exchange Class                    |
    │                              .                              |
    +=============================================================+
```

当前版本中所有的公开的，私有的api均以http方式能提供，WebSocket和FIX正在开发中...,我们已经支持116个交易所

除了一些基本的交易所和交易数据之外，一些交易所提供保证金交易（杠杆），各种衍生品（如期货合约和期权），还有暗池，柜台交易（OTC），商户API等等。

### 初始化交易所

 ```python
# Python
import ccxt
exchange = ccxt.okcoinusd () # default id
okcoin1 = ccxt.okcoinusd ({ 'id': 'okcoin1' })
okcoin2 = ccxt.okcoinusd ({ 'id': 'okcoin2' })
id = 'btcchina'
btcchina = eval ('ccxt.%s ()' % id)
gdax = getattr (ccxt, 'gdax') ()
 ```

### 交易所结构

所有的交易所都有一套基本的属性和方法，大多数情况你可以通过重写构造方法来实现，当然，你也可以通过子类重写。

```json
{
    'id':   'exchange'                  // lowercase string exchange id
    'name': 'Exchange'                  // human-readable string
    'countries': [ 'US', 'CN', 'EU' ],  // string or array of ISO country codes
    'urls': {
        'api': 'https://api.example.com/data',  // string or dictionary of base API URLs
        'www': 'https://www.example.com'        // string website URL
        'doc': 'https://docs.example.com/api',  // string URL or array of URLs
    },
    'version':         'v1',            // string ending with digits
    'api':             { ... },         // dictionary of api endpoints
    'has': {                            // exchange capabilities
        'CORS': false,
        'publicAPI': true,
        'privateAPI': true,
        'cancelOrder': true,
        'createDepositAddress': false,
        'createOrder': true,
        'deposit': false,
        'fetchBalance': true,
        'fetchClosedOrders': false,
        'fetchCurrencies': false,
        'fetchDepositAddress': false,
        'fetchMarkets': true,
        'fetchMyTrades': false,
        'fetchOHLCV': false,
        'fetchOpenOrders': false,
        'fetchOrder': false,
        'fetchOrderBook': true,
        'fetchOrders': false,
        'fetchTicker': true,
        'fetchTickers': false,
        'fetchBidsAsks': false,
        'fetchTrades': true,
        'withdraw': false,
    },
    'timeframes': {                     // empty if the exchange !has.fetchOHLCV
        '1m': '1minute',
        '1h': '1hour',
        '1d': '1day',
        '1M': '1month',
        '1y': '1year',
    },
    'timeout':          10000,          // number in milliseconds
    'rateLimit':        2000,           // number in milliseconds
    'userAgent':       'ccxt/1.1.1 ...' // string, HTTP User-Agent header
    'verbose':          false,          // boolean, output error details
    'markets':         { ... }          // dictionary of markets/pairs by symbol
    'symbols':         [ ... ]          // sorted list of string symbols (traded pairs)
    'currencies':      { ... }          // dictionary of currencies by currency code
    'markets_by_id':   { ... },         // dictionary of dictionaries (markets) by id
    'proxy': 'https://crossorigin.me/', // string URL
    'apiKey':   '92560ffae9b8a0421...', // string public apiKey (ASCII, hex, Base64, ...)
    'secret':   '9aHjPmW+EtRRKN/Oi...'  // string private secret key
    'password': '6kszf4aci8r',          // string password
    'uid':      '123456',               // string user id
}
```

### 交易所属性

下面描述的所有属性为基础交易所属性:

- id：区分交易所平台。
- name: 交易所名字。
- countries：交易所所在国家。
- urls['api']：交易所API数组。
- urls['www']:交易所主页。
- urls['doc']：API文档链接。
- version: 版本。
- api: 
- has：拥有的交换功能。
- timeframes: 时间框架
- timeout: 超时时间(默认为10s)、
- rateLimit: 调用时间差。
- enableRateLimit: 是否启用调用限制。
- userAgent 
- verbose:是否启用日志。
- markets: 
- symbols: 支持币种
- currencies: 货币
- markets_by_id 
- proxy 
- apiKey 
- secret 
- password 
- uid 

## 关于访问限制

- 保持在交易所限制范围内。
- 如果交易所使用了ddos保护，使用[绕过限制](https://github.com/ccxt/ccxt/blob/master/examples/py/bypass-cloudflare.py)
- 使用代理
- 与交易所交流，开启你的IP白名单。
- 做爬虫集群
- 使用多个IP
- ......

如果你被唧唧了，cctx会抛出异常:

- DDoSProtectionError
- ExchangeNotAvailable
- ExchangeError

## 加载市场

```python
# Python
okcoin = ccxt.okcoinusd ()
markets = okcoin.load_markets ()
print (okcoin.id, markets)
```

### 币标志和id

```python
# Python

print (exchange.load_markets ())

etheur1 = exchange.markets['ETH/EUR']      # get market structure by symbol
etheur2 = exchange.market ('ETH/EUR')      # same result in a slightly different way

etheurId = exchange.market_id ('BTC/USD')  # get market id by symbol

symbols = exchange.symbols                 # get a list of symbols
symbols2 = list (exchange.markets.keys ()) # same as previous line

print (exchange.id, symbols)               # print all symbols

currencies = exchange.currencies           # a list of currencies

kraken = ccxt.kraken ()
kraken.load_markets ()

kraken.markets['BTC/USD']                  # symbol → market (get market by symbol)
kraken.markets_by_id['XXRPZUSD']           # id → market (get market by id)

kraken.markets['BTC/USD']['id']            # symbol → id (get id by symbol)
kraken.markets_by_id['XXRPZUSD']['symbol'] # id → symbol (get symbol by id)
```

