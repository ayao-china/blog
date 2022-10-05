---
title: Kaggle Titanic
date: 2022-10-05 10:25:03
sticky: true
categories:
- [数据分析]
top_img: http://tva1.sinaimg.cn/large/006UraCCgy1h6ul740vptj31hc0sndyv.jpg
cover: http://tva1.sinaimg.cn/large/006UraCCgy1h6ul740vptj31hc0sndyv.jpg
---
# kaggle 泰坦尼克号
## 前言
kaggle入门竞赛 泰坦尼克号：https://www.kaggle.com/competitions/titanic  
数据分析工具 python jupyter-notebook sklearn
## 整理数据
导入数据，使用pandas对数据进行初步处理
```python

import pandas as pd
from sklearn.preprocessing import LabelEncoder,OrdinalEncoder,OneHotEncoder
from sklearn.preprocessing import StandardScaler,MinMaxScaler
Train = pd.read_csv('./modelfile/train.csv',encoding='utf-8')
Test = pd.read_csv('./modelfile/test.csv',encoding='utf-8')
res = pd.read_csv('./modelfile/gender_submission.csv',encoding='utf-8')
# 数据合并
tmp = Train.drop(columns='Survived')
total_data = pd.concat([tmp,Test])
LE = LabelEncoder()
St = StandardScaler()
```
+ 查看各个字段的缺失情况，age fare cabin都有缺失。在补充数据之前，应该先确定这些字段受哪些字段的影响比较大，根据其他字段的分布情况合理补充缺失数据。  
+ 使用相关性分析，查看age与其他字段的相关性程度，主观因素首先猜测年龄与船舱等级class,港口Embarked，以及票价Fare等相关，其他无关因素排除
{% asset_img 1664955184914.png 相关性分析 %}
class与age负相关且系数最大，则选取地位等级class作为其中一个影响因素；姓名name数据较全，每个名字中都含有其尊称 可以从title入手。提取所有名字的title,以title和class做分组，使用分组中位数补充年龄缺失字段
```python

total_data['title'] = total_data['Name'].apply(lambda n:str(n).split(",")[1].split(".")[0].replace(' ','').replace('Ms','Miss'))
age_mean = Train.groupby(['Pclass','title']).median()['Age'].reset_index()
total_data['Age'].fillna(0, inplace=True)
total_data = pd.merge(total_data, age_mean, 'left', on=['Pclass','title'])
total_data['Age'] = total_data.apply(lambda x: x['Age_x'] if x['Age_x'] > 0 else x['Age_y'], axis=1)
```
+ 票价fare从主观判断来看与地位等级class和港口Embarked相关，地位等级越高票价消费越高；港口代表了里程距离，里程越大票价肯定会越高，以港口和地位分组取中位数补充票价缺失值，并将票价转为标准值
```python

fare = Train.groupby(['Pclass','Embarked']).median()['Fare'].reset_index()
total_data['Fare'].fillna(0.1,inplace=True)
total_data = pd.merge(total_data, fare, 'left', on=['Pclass', 'Embarked'])
total_data['Fare'] = total_data.apply(lambda x: x['Fare_x'] if x['Fare_x'] > 0.1 and x['Fare_x'] == 0 else x['Fare_y'], axis=1)
total_data['Fare'] = St.fit_transform(np.array(Train['Fare'].values.tolist()).reshape(-1,1))

```
+ 船舱号cabin字段缺失较多，可以考虑去除不分析。
## 分析数据
+ 通过提供的train文件查看每个字段与幸存与否的情况。
+ 地位等级class与幸存概率正相关，地位越高的人所受的待遇越好幸存几率越大。  {% asset_img 1664961918240.png 幸存者与遇难者的等级分布情况 %}
+ 性别sex也可影响幸存几率，男士生理结构的先天优势可能会使其幸存的概率更高，但是鉴于历史文化背景，女士优先的传统也可能提高女性的生存几率。
+ 姓名name中含有title，比如教授或者贵族，这些与地位class的影响相似，包含了其社会地位或者年龄阶层，也可算做一个影响因素。 将title和sex标准化处理
```python

total_data['Sex'] = LE.fit_transform(total_data['Sex'].values.tolist())
total_data['title'] = LE.fit_transform(total_data['title'].values.tolist())
```
+ SibSp 和Parch所提供的信息都是同行者人数，可以放在一起分析，从幸存与遇难者的同行者数量分布来看，有一到两个同行者的情况下幸存的概率更大一些，可能是同行者数量更多会争夺求生资源。所以这个数据可分为同行者数量等于0 ，等于1或者2,大于2这三种情况，并做标准化处理{% asset_img 1664962525732.png 幸存者与遇难者的同行者分布情况 %}
```python

# 同行者数量分布
total_data['family'] = total_data['SibSp'] + total_data['Parch']
total_data['family'] = total_data['family'].apply(lambda x:'a' if x in (1,2) else 'b' if x ==3 else 'c')
total_data['family'] = LE.fit_transform(Train['family'].values.tolist())
```
+ 座位号Ticket一部分有字母，一部分只有数字，猜测座位分布影响了幸存概率，可以将其进一步处理，提取座位号中的首字母，根据首字母将数据标准化
```python

# 座位号分布查看
total_data['seat'] = total_data['Ticket'].str.extract('([A-Za-z]+)',expand=True)
total_data['seat'].fillna('n', inplace=True)
total_data['cab'] = total_data['Ticket'].apply(lambda c: str(c).split("/")[0] if "/" in str(c) else 'n')
total_data['cab'] = total_data.apply(lambda t: t['cab'] if t['cab'] != 'n' else 's' if t['seat'] != 'n' else 'num',axis=1)
total_data['cab'] = LE.fit_transform(Train['cab'].values.tolist())
```
票价fare已经在前期处理成了标准值，可以直接放入模型。
Embarked 港口号提供的信息有限，既不是数值，也没有直接信息表明它与求生者是否幸存有关，可以去除不分析。
## 模型预测
+ 使用k近邻和随机森林作为预测模型，交叉验证评估模型性能，并寻找最优参数。从评估结果来看，K近参数为8时评分最高0.796，随机森林评分最高0.838，因此使用随机森林做预测结果会更准确

```python

Train.set_index('PassengerId',inplace=True)
x_train = np.array(total_data[:891])
y_train= np.array(Train['Survived'])
x_test = np.array(total_data[891:])

# K近邻
sn = 0
tn = 0
for ne in range(4,10):
    KNN = KNeighborsClassifier(n_neighbors=ne)
    KNN_model = KNN.fit(x_train,y_train)
    sroce1 = cross_val_score(KNN, x_train, y_train, cv=8)
    print(ne,sroce1.mean())
    if sn < sroce1.mean():
        tn = ne
        sn = sroce1.mean()
    else:
        continue
print(tn,sn)
# 随机森林
from sklearn.ensemble import RandomForestClassifier
# 交叉验证评估模型性能
from sklearn.model_selection import cross_val_score
from sklearn import svm

# 查找最优参数
t = (5,54)
sm = 0
for dep in range(4,8):
    for num in range(30,60):
        RF = RandomForestClassifier(max_depth=dep,n_estimators=num)
        RF.fit(x_train, y_train)
        clf = svm.SVC(kernel='linear', C=1)
        sroce2 = cross_val_score(RF,x_train,y_train,cv=8)
        if sm < sroce2.mean():
            t = (dep,num)
            sm = sroce2.mean()
        else:
            continue
RF = RandomForestClassifier(max_depth=t[0],n_estimators=t[1])
RF.fit(x_train, y_train)
Test['Survived'] = RF.predict(x_test)
Test.reset_index(inplace=True)
Test = Test[['PassengerId','Survived']]
res = pd.merge(res[['PassengerId']],Test,'left',on=['PassengerId'])

```