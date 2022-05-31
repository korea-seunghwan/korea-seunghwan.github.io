
```
    conda create -n test python=3.8
    pip install sklearn
    pip install pandas
    pip install quandl
```

tutorial은 https://www.youtube.com/watch?v=JcI5Vnw0b2c&list=PLQVvvaa0QuDfKTOs3Keq_kaG2P55YRn5v&index=2 여기의 tutorial을 토대로 정리해보고자 한다.

먼저 새로운 가상환경을 설정하고 위의 library들을 설치해주자. quandl에서의 데이터를 활용하여 공부해보자.

## Regression Intro - Practical Machine Learning Tutorial with Python

```python

    import pandas as pd
    import quandl

    df = quandl.get("WIKI/GOOGL")

    df = df[["Adj. Open", "Adj. High", "Adj. Close", "Adj. Volume"]]
    df['HL_PCT'] = (df['Adj. High'] - df['Adj. Close']) / df['Adj. Close'] * 100.0
    df['PCT_change'] = (df['Adj. Close'] - df['Adj. Open']) / df['Adj. Open'] * 100.0

    df = df[['Adj. Close', 'HL_PCT', 'PCT_change', 'Adj. Volume']]

    print(df.head())

```

GOOGL의 주가를 가지고와서 하루에 올라갔던 변동폭과 시작할때와 장을 마감할때의 가격의 변동폭을 구해보는 code이다.

## Regression Features and Labels - Practical Machine Learning Tutorial with Python

주어진 데이터를 shift를 활용해서 얼마나 가격변동이 있었는지 알아보자.

```python

    forecast_col = 'Adj. Close'
    df.fillna(-99999, inplace=True)

    forecaset_out = int(math.ceil(0.01*len(df)))

    df['label'] = df[forecast_col].shift(-forecaset_out)
    df.dropna(inplace=True)

    print(df.head())

```

## Regression Training and Testing - Practical Machine Learning Tutorial with Python

실제로 모델을 fit 시켜서 test dataset에 대해 성능을 측정해보자.
주어진 데이터를 X_train, X_test로 나눠서 train data는 training에, test data는 실제 성능을 확인하는데 사용한다.

```python

    import math, quandl
    import pandas as pd
    import numpy as np
    from sklearn import preprocessing, svm
    from sklearn.model_selection import train_test_split
    from sklearn.linear_model import LinearRegression

    df = quandl.get("WIKI/GOOGL")

    df = df[["Adj. Open", "Adj. High", "Adj. Close", "Adj. Volume"]]
    df['HL_PCT'] = (df['Adj. High'] - df['Adj. Close']) / df['Adj. Close'] * 100.0
    df['PCT_change'] = (df['Adj. Close'] - df['Adj. Open']) / df['Adj. Open'] * 100.0

    df = df[['Adj. Close', 'HL_PCT', 'PCT_change', 'Adj. Volume']]

    forecast_col = 'Adj. Close'
    df.fillna(-99999, inplace=True)

    forecast_out = int(math.ceil(0.01*len(df)))
    print(forecast_out)

    df['label'] = df[forecast_col].shift(-forecast_out)
    df.dropna(inplace=True)

    X = np.array(df.drop(['label'],1))
    y = np.array(df['label'])
    X = preprocessing.scale(X)
    y = np.array(df['label'])

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

    clf = LinearRegression(n_jobs=-1)
    # clf = svm.SVR(kernel='poly')
    clf.fit(X_train, y_train)
    accuracy = clf.score(X_test, y_test)

    print('accuracy: ', accuracy)

```