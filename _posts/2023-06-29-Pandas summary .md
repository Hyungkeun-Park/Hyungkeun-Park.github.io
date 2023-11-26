---
title: "Pandas 기본 정리"
excerpt: "자꾸 까먹어서 기본 정리해두기"

categories:
    - Pandas
tags:
    - Pandas
last_modified_at: 2023-11-26T16:52:00
---

# Pandas
Python을 이용해 데이터를 만지려는 사람들이라면 누구나<br>
마주하게될 `Pandas`라는 라이브러리<br>

사용할 때는 자꾸 사용하니까 까먹지 않는데<br>
가끔씩 사용하는 나로서는 쓸때마다 자꾸 새로워서<br>

딱 기본적으로 사용하는데 필요한 애들만 조금 정리해두려고 한다<br>

<br>
<br>

# DataFrame 생성
---
기본적인 데이터 프레임 생성은 다음과 같이 한다
```python
import pandas as pd
df=pd.DataFrame(data=, index=, columns=,)
```
그런데 종종 시험을 치다 보면 꼭 index가 1부터 시작해야한다는 조건을 붙이곤 한다<br>
그럴땐 간단한게
```python
df.index=df.index+1
```
위와 같이 해주면 0부터 시작하던 index를 1부터 시작시킬 수 있다

<br>
<br>

# DataFrame 데이터 접근
---
특정 column 또는 row에 접근하는 방법은 다음과 같다
```python
df.loc['column_name'] # 원하는 column 한 개에 접근
df.loc[['column_name1','column_name2']] # 원하는 다수의 column 접근
df.iloc[num] # 원하는 num 번째의 row에 접근
df.iloc[:,num] #원하는 num 번째의 column 데이터 전체에 접근 

df.head(num) # DataFrame의 앞에서 num개의 데이터에 접근
df.tail(num) # DataFrame의 뒤에서 num개의 데이터에 접근

df.select_dtypes(include="object") # column 데이터 타입이 object인 데이터에 접근
df.select_dtypes(exclude="object") # column 데이터 타입이 object가 아닌 데이터에 접근
```
<br>
<br>

# DataFrame 정보 접근
---
데이터를 다루다 보면 종종 결측치를 마주할 경우가 있고<br>
데이터들에 대한 기본 통계량 등을 확인할 필요가 있다
```python
df.info() # 각 컬럼 데이터 수 + 결측치 수 및 타입을 확인 가능

df.describe() # 평균, 편차, 사분위수, 최대, 최소값 확인 가능

df['column_name'].unique() # column에 대한 유일값 종류 확인 가능
df['column_name'].nunique() # column에 대한 유일값 개수 확인 가능
```
<br>
<br>

# 특정 조건 만족 데이터 접근 및 변경
---
특정 조건을 만족하는 데이터에 접근하는 방법은 다음과 같다
```python
condition=원하는 조건을 입력
df[condition] #원하는 조건을 만족하는 데이터 출력

#Ex)
condition=df['강수량']>5 # '강수량' column 데이터 중 값이 5 초과인 데이터에 대해 bool list가 반환
print(df.loc[condition]) # 조건에 만족하는(True) 데이터들에 접근

#만약 두 가지 이상의 조건에 대해 접근하고자 한다면
condition=조건1 & 조건2 # 두 조건을 동시 만족
codnition=조건1 | 조건2 # 두 조건 중 한 가지만 만족하더라도

# Ex)
df.loc[condition,'강수량']+=10 # 강수량이 5 초과인 데이터의 강수량 값에 10을 더한 값을 넣어준다
```

<br>
<br>

# 특정 column 속성 변경
---
특정 column에 대한 type을 변경하거나 column을 추가 제거하는 방법은 다음과 같다
```python
df['column name']=추가 데이터 # 원하는 데이터를 특정 column에 추가

df.drop('column name',axis=1) # 원하는 column 제거
del df['column name'] # 원하는 column 제거

df['column name'].astype({'column name':int}) # 정수로 해당 column 속성 변경
```

<br>

그리고 금액 데이터가 데이터프레임의 경우<br>
종종 콤마가 들어가 처리가 귀찮은 경우가 생기는데<br>
다음과 같이 간단하게 처리가 가능하다<br>
```python
df=pd.read_csv("data.csv",thousand=',') # 콤마게 제거된 데이터로 read
df.loc['금액']=df['금액'].str.replace(',','').astype('int64')
```
<br>
<br>
<br>
위 기능들만 기본적으로 외워두고 짬뽕시킨다면<br>
원하는 데이터에 어지간해선 접근이 가능하다<br>
<br>
<br>
~~제발 까먹지좀 말자...~~