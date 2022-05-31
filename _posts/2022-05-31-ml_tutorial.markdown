---
layout: post
title: "Machine Learning tutorial"
subtitle: "Python Machine Learning Tutorial (Data Science)"
date: 2022-05-31 00:00:00 +0900
tag: machine learning
background: "/img/bg-post.jpg"
---

Machine Learning과 Deep learning의 경계, 혹은 실무에서는 왜 ML Engineering을 더 많이 뽑는지 하는일이 뭐가 다른지 알아보기 위해 tutorial을 하나 진행해보려고 한다.

tutorial은 <https://youtu.be/7eh4d6sabA0> 을 보고 따라해본다.

### Machine Learning in Action

* Import the Data
* Clean the Data
* Split the Data into Training/Test Sets
* Create a Model
* Train the Model
* Make Predictions
* Evaluate and Improve

### Libraries and Tools

Numpy, Pandas, MatPlotLib, Scikit-Learn 이 주요 사용 Library이다

Anaconda 설치 및 사용법은 따로 다루지 않겠다.

jupyter notebook도 설치해준다

![실행결과][/img/machine_learning_tutorial/1.png]

### Importing a DataSet

kaggle에서 video games sales 검색해서 data를 다운로드 받자

![실행결과2][/img/machine_learning_tutorial/2.png]

### A Real Machine Learning Problem

동영상 아래 링크에서 데이터를 다운로드 받을 수 있다. music.csv파일을 다운로드 받아서 data를 준비해주자.

![실행결과3][/img/machine_learning_tutorial/3.png]

### Preparing the Data

실제 load한 데이터를 input과 output으로 나누어주는 작업이 필요하다. 어떤 column을 output으로 볼 것인가 정해주는 수순이다.

```python
    X = music_data.drop(column='genre')
    y = music_data['genre]
```

### Learning and Predicting

X를 fit 시켜 y를 예측하게 할 모델을 선정해준다.

```python
    from sklearn.tree import DecisionTreeClassifier

    model = DecisionTreeClassifier()
    model.fit(X, y)

    predictions = model.predict([[21, 1], [22, 0]])
```

위에서 training으로 사용할 데이터를 fit 시킨다. age와 genre의 상관관계를 활용하여 선호하는 music genre를 맞추는 모델이다. 따라서 21살의 남자의 경우를 예측해본 예제이다.

### Calculating the Accuracy

모든 데이터를 학습 시키면 test를해서 accuracy를 계산할 수 없기 때문에 가진 데이터의 80%를 training에, 20%를 test에 사용한다.

```python
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import accuracy_score

    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
    
    model.fit(X_train, y_train)
    predictions = model.predict(X_test)
    score = accuracy_score(y_test, predictions)
```

가진 데이터의 80%만 사용해서 나머지로 test해본 것이다. 나눠지는 데이터에 따라 fit하는 정도가 달라지기 때문에 정확도는 실행할 때 마다 달라질 수 있다.

### Persisting Models

fit 시킨 model을 저장하고 불러와서 prediction해보자

```python
    import pandas as pd
    from sklearn.tree import DecisionTreeClassifier
    import joblib

    # music_data = pd.read_csv('../data/ml_tutorial/music/music.csv')
    # X = music_data.drop(columns=['genre'])
    # y = music_data['genre']

    # model = DecisionTreeClassifier()
    # model.fit(X, y)

    # joblib.dump(model, 'music-recommender.joblib')
    model = joblib.load('music-recommender.joblib')
    predictions = model.predict([[21,1]])
    predictions
```

deep learning에서 모델 저장하고 불러오는거랑은 완전히 다른 느낌이다. 훨씬 간결하고 직관적이라 이해가 빨리 된다.

### Visualizing Decision Trees

```python
    import pandas as pd
    from sklearn.tree import DecisionTreeClassifier
    from sklearn import tree

    music_data = pd.read_csv('../data/ml_tutorial/music/music.csv')
    X = music_data.drop(columns=['genre'])
    y = music_data['genre']

    model = DecisionTreeClassifier()
    model.fit(X, y)

    tree.export_graphviz(model, out_file='music-recommenter.dot', 
                        feature_names=['age', 'gender'], 
                        class_names=sorted(y.unique()),
                        label='all',
                        rounded=True,
                        filled=True)
```

![실행결과3][/img/machine_learning_tutorial/4.png]
