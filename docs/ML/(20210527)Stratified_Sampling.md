---
layout: index
title: Stratified_Sampling모형 최적화
nav_order: 3
parent: ML
permalink: /ML2
---

작성일자: 2021/05/27

## Stratified Sampling(층화추출법)

* Class imbalance 문제에서 한 minority class의 f1-score를 높이기 위해 층화추출법을 사용



정의

- 모집단을 중복되지 않도록 층으로 나눈 후 각 층에서 표본을 추출하는 표본추출 방법론



과정

1) 층의 개수, 층간 분할 기준 설정

- 정답 데이터의 class를 기준으로 층을 분할

2) 각 클래스에 대해 모집단과 동일한 비율로 표본을 추출



코드 리뷰



```python
from sklearn.model_selection import train_test_split
# Y는 
x_train, x_test, y_train, y_test = train_test_split(X,Y, test_size=0.2,
                                                   stratify=Y)
```



* 참고

층화추출은 Cross-Validation에서도 사용 가능

```python
from sklearn.model_selection import StratifiedKFold
# 3-fold cross validation
skf = StratifiedKFold(n_splits=3)
skf.split(X,Y)

n_iter = 0
for train_index, test_index in skf.split(X,y):
    n_iter += 1
    label_train = y.iloc[train_index]
    label_test = y.iloc[test_index]
    print("교차검증 번호: {0}".format(n_iter))
    print("학습 레이블 분포")
    print(label_train.value_counts())
    print("검증 레이블 분포")
    print(label_test.value_counts())
    print("")
```

n_splits(3번)만큼 교차검증 -> 레이블 별 분포가 동일

```
>>>
교차검증 번호: 1
학습 레이블 분포
no     2434
yes     399
Name: churn, dtype: int64
검증 레이블 분포
no     1218
yes     199
Name: churn, dtype: int64

교차검증 번호: 2
학습 레이블 분포
no     2435
yes     398
Name: churn, dtype: int64
검증 레이블 분포
no     1217
yes     200
Name: churn, dtype: int64

교차검증 번호: 3
학습 레이블 분포
no     2435
yes     399
Name: churn, dtype: int64
검증 레이블 분포
no     1217
yes     199
Name: churn, dtype: int64
```



Reference



1. https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html
2. https://velog.io/@skyepodium/K-Fold-%EA%B5%90%EC%B0%A8%EA%B2%80%EC%A6%9D