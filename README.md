WIP: I will update this demo occasionally for new functionality.
This demo program is to use [gdax python client](https://github.com/danpaquin/gdax-python) to access one's own gdax account to get account balance and setup frequent short-term order.
```bash
# First to install gdax python client package
pip install gdax
```

1. setup gdax API with certain permissions. [here](https://www.gdax.com/settings/api)
2. paste generarted API key, secret, and passphrase to the code.
3. You are good to go.

Disclaimer: Please revise your code before placing any order. I am not responsible for any financial loss because of the usage of the code.

```python
import gdax
import time

key = 'xxx'
b64secret = 'yyy'
passphrase = 'zzz'
_IS_BUY = True
_IS_SELL = False

auth_client = gdax.AuthenticatedClient(key, b64secret, passphrase)

def get_last_price_operation(client):
    last_fill = client.get_fills(product_id="BTC-USD")[0][0]
    last_price = float(last_fill['price'])
    last_operation = _IS_BUY if last_fill['side'] == 'buy' else _IS_SELL
    return last_price, last_operation

def get_account_balance(client):
    my_accounts = client.get_accounts()
    usd = None
    eth = None
    btc = None
    ltc = None
    bch = None
    for my_account in my_accounts:
        if my_account['currency'] == 'USD':
            usd = float(my_account['balance'])
        elif my_account['currency'] == 'LTC':
            ltc = float(my_account['balance'])
        elif my_account['currency'] == 'BTC':
            btc = float(my_account['balance'])
        elif my_account['currency'] == 'ETH':
            eth = float(my_account['balance'])
        elif my_account['currency'] == 'BCH':
            bch = float(my_account['balance'])
    btc_price = float(client.get_product_ticker('BTC-USD')['price'])
    ltc_price = float(client.get_product_ticker('LTC-USD')['price'])
    eth_price = float(client.get_product_ticker('ETH-USD')['price'])
    bch_price = float(client.get_product_ticker('BCH-USD')['price'])

    account_summary = usd + btc_price * btc + ltc_price * ltc + eth_price * eth + bch_price * bch
    print('my account balance is {0:.2f} usd'.format(account_summary))
    return usd, eth, btc, ltc, bch

def last_24_hr_mid_price(client, product_id='BTC-USD'):
    return (float(client.get_product_24hr_stats(product_id)['high'])
            + float(client.get_product_24hr_stats(product_id)['low'])) / 2.0

exchange_amount = 0.05
notify_margin = 0.01
profit_margin = notify_margin * 0.5
print('exchange_amount:{:.4f}, notify_margin:{:.4f}, profit_margin:{:.4f}'.format(exchange_amount,
                                                                                  notify_margin, profit_margin))

mid = last_24_hr_mid_price(auth_client)

print('last 24 hrs mid: {}'.format(mid))

while True:
    last, operation = get_last_price_operation(auth_client)
    operation = not operation
    print('last filled btc price is {}'.format(last))
    curr_btc_price = float(auth_client.get_product_ticker('BTC-USD')['price'])
    time.sleep(5)
    print('current btc price is {}'.format(curr_btc_price))
    curr_usd, _, curr_btc, _, _ = get_account_balance(auth_client)
    time.sleep(5)
    if curr_btc_price > (last * (1 + notify_margin)) and operation == _IS_SELL:
        if curr_btc > exchange_amount:
            selling_price = curr_btc_price * (1 + profit_margin)
            print('trying to sell {:.3f} btc @ {:.2f}'.format(exchange_amount, selling_price))
            response = auth_client.sell(price='{:.2f}'.format(selling_price),
                                        size='{:.3f}'.format(exchange_amount),
                                        product_id='BTC-USD')
        else:
            print('insufficient btc')
    elif curr_btc_price < (last * (1 - notify_margin)) and operation == _IS_BUY:
        assert (1 - profit_margin) > 0
        if curr_usd > (exchange_amount * curr_btc_price * (1 - profit_margin)):
            buying_price = curr_btc_price * (1 - profit_margin)
            print('trying to buy {:.3f} btc @ {:.2f}'.format(exchange_amount, buying_price))
            response = auth_client.buy(price='{:.2f}'.format(buying_price),
                                       size='{:.3f}'.format(exchange_amount),
                                       product_id='BTC-USD')
        else:
            print('insufficient usd')
    time.sleep(15)
```
