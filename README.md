# stock-sh-csv

### 包含了A股上海证券交易所，所有股票所有时间段的日频k线csv数据

事先下载好的上交所日线csv数据。

下载：点github界面右上角的绿色<>Code按钮，然后选菜单下面的Download Zip就行，用git也可以。因为只有上交所没有深交所，所以只有大约1685支股票，解压之后一共大约有150MB。

来源：数据都是直接使用akshare库的stock_zh_a_hist下载后保存到csv的。stock_zh_a_hist会用这个网站爬取数据：<https://quote.eastmoney.com/concept/sh603777.html?from=classic>。都是前赋权的数据。

用法：直接用pandas读取csv文件。比如：

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import os
import random

# 随机绘制一支股票
def plot_random_stock():
    filename = random.choice(os.listdir('stocks'))
    df = pd.read_csv('stocks/' + filename)
    plt.plot(df['close'])
    plt.plot(df['close'].rolling(5).mean())
    plt.plot(df['close'].rolling(20).mean())
    plt.show()

# 读取所有股票最后一天的收盘价
def read_all_stocks():
    for filename in os.listdir('stocks'):
        df = pd.read_csv('stocks/' + filename)
        print(filename, df.iloc[-1, 'close'])
```

重新下载数据:

```python
import pandas as pd
import akshare as ak
import os
def download_all_stocks():
    print('start')
    # create `stocks` folder
    if not os.path.exists('stocks'):
        os.mkdir('stocks')
    df = get_name_code()
    df.to_csv('stocks/name_code.csv')
    for i in range(len(df)):
        symbol = df.index[i]
        if os.path.exists(f'stocks/{symbol}.csv'):
            continue
        print(f'{i}/{len(df)} {symbol}')
        stock = get_stock(symbol)
        stock.rename(columns={
            '日期': 'date',
            '开盘': 'open',
            '收盘': 'close',
            '最高': 'high',
            '最低': 'low',
            '成交量': 'volume',
        }, inplace=True)
        stock.drop(columns=['成交额', '振幅', '涨跌幅', '涨跌额', '换手率'], inplace=True)
        stock.set_index('date', inplace=True)
        stock.to_csv(f'stocks/{symbol}.csv')
    print('done')
```

你的python程序需要能访问到stocks文件夹。所以你可以把python脚本和stocks文件夹放在同一个目录下。

文件结构：

```
test.py
download.py
stocks
├── 000001.csv
├── 000002.csv
├── ...
└── 603999.csv
```
