# stock-sh-csv

### 包含了A股上海证券交易所，所有股票所有时间段的日频k线csv数据

事先下载好的上交所日线csv数据。

下载：点github界面右上角的绿色`<>Code`按钮，然后选菜单下面的Download Zip就行，用Git也可以。因为只有上交所没有深交所，所以只有大约1685支股票，解压之后一共大约有150MB。

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

你的Python程序需要能访问到stocks文件夹。所以你可以把Python脚本和stocks文件夹放在同一个目录下。

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

除了像这样一次性下载所有数据，也可以试一试用joblib.Memory把下载的数据缓存到cache文件夹，防止它每次运行都重复下载。

和别的技术相比，金融这方面的库、教程、信息相对都比较少。根据我的尝试，akshare是能找到的相关的库中最方便的一个。比别的库反应快不少，而且不需要登陆。数据的类型挺多的，而且读出来就直接是整理好类型的Pandas DataFrame。除此之外，还可以看一下yfinance，可以从雅虎上面爬取美股一类的数据。还可以去Kaggle上学学机器学习。聚宽一类的线上平台，或者backtrader，支持更复杂的模拟交易和回测。talib提供了一些技术指标、因子。如果能用万得一类的工具的话，能获得很多信息，还能直接下载研究报告，这是最好的。


