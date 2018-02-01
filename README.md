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

auth_client = gdax.AuthenticatedClient(key, b64secret, passphrase)

def get_account_balance(auth_client):
    my_accounts = auth_client.get_accounts()
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
    btc_price = float(auth_client.get_product_ticker('BTC-USD')['price'])
    ltc_price = float(auth_client.get_product_ticker('LTC-USD')['price'])
    eth_price = float(auth_client.get_product_ticker('ETH-USD')['price'])
    bch_price = float(auth_client.get_product_ticker('BCH-USD')['price'])

    account_summary = usd + btc_price * btc + ltc_price * ltc + eth_price * eth + bch_price * bch
    print('my acount balance is {0:.2f} usd'.format(account_summary))
    return usd, eth, btc, ltc, bch


exchange_amount = 0.001
notify_margin = 0.01
profit_margin = notify_margin * 0.5
print('exchange_amount:{:.4f}, notify_margin:{:.4f}, profit_margin:{:.4f}'.format(exchange_amount,
                                                                                  notify_margin, profit_margin))

mid = (float(auth_client.get_product_24hr_stats('BTC-USD')['high'])
       + float(auth_client.get_product_24hr_stats('BTC-USD')['low'])) / 2.0
last = mid
print('last 24 hrs mid: {}'.format(last))
buytoggle = False
while True:
    print('last btc price is {}'.format(last))
    curr_btc_price = float(auth_client.get_product_ticker('BTC-USD')['price'])
    time.sleep(5)
    print('current btc price is {}'.format(curr_btc_price))
    curr_usd, _, curr_btc, _, _ = get_account_balance(auth_client)
    time.sleep(5)
    if curr_btc_price > (last * (1 + notify_margin)) and not buytoggle:
        if curr_btc > exchange_amount:
            selling_price = curr_btc_price * (1 + profit_margin)
            print('trying to sell {:.6f} btc @ {:.6f}'.format(exchange_amount, selling_price))
            auth_client.sell(price=str(curr_btc_price * (1 + profit_margin)),
                             size=str(exchange_amount),
                             product_id='BTC-USD')
            last = curr_btc_price
            buytoggle = True
        else:
            print('insufficient btc')
    elif curr_btc_price < (last * (1 - notify_margin)) and buytoggle:
        assert (1 - profit_margin) > 0
        if curr_usd > (exchange_amount * curr_btc_price * (1 - profit_margin)):
            buying_price = curr_btc_price * (1 - profit_margin)
            print('trying to buy {:.6f} btc @ {:.6f}'.format(exchange_amount, buying_price))
            auth_client.buy(price=str(curr_btc_price * (1 - profit_margin)),
                            size=str(exchange_amount),
                            product_id='BTC-USD')
            last = curr_btc_price
            buytoggle = False
        else:
            print('insufficient usd')
    time.sleep(15)


```
