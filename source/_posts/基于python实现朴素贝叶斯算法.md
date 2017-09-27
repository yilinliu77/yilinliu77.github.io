---
title: 基于python实现朴素贝叶斯算法
date: 2017-08-30 09:35:25
tags:
- Machine Lerning
- Python
- Computer Vision
categories: Machine Learning
---

# 朴素贝叶斯算法总结

(有关朴素贝叶斯算法的原理可可参考[算法杂货铺——分类算法之朴素贝叶斯分类(Naive Bayesian classification)](http://www.cnblogs.com/leoo2sk/archive/2010/09/17/naive-bayesian-classifier.html))

下面对朴素贝叶斯算法进行粗略的总结，并可以根据这个总结搭建朴素贝叶斯算法的框架

首先，将朴素贝叶斯算法分为`数据预处理->训练->识别三个模块`，以下是完成Titanic预测后对算法的总结，**对应的算法实现部分都有相应的实例**

//Tasking图

在数据预处理部分，我们对原始数据进行预处理，将数据转化为自己想要的格式，例如
- 对一些属性及结果进行划分
- 字符串->可操作格式(例如int)
- 抛弃不需要的属性值
- 填补空缺值
在完成数据预处理后，预期得到
- `dataMatrix`矩阵：存储模型所需要数据的值，横向为不同的属性，纵向为不同的记录。
- `cateVec`向量：存储分类结果，与`dataMatrix`的记录一一对应，也就是说`cateVec`的长度与`dataMatrix`的记录数相等

在训练部分，我们将`dataMatrix`与`cateVec`输入，并计算每一个划分的先验概率`P（ai=j|C=k）`并进行分类
- 通过计算一个属性的一个划分在这个类别中出现的次数除以这个类别出现的总次数得到划分的先验概率，也就是`P（ai=j|C=k）`
- 计算类别的概率`P（C=k）`
在第一点中，我们期望得到的应该是**所有属性的所有划分的概率**，例如我们两个属性一个有两个划分，一个有三个划分，那一共期望的输出应该是5个概率
期望输出：
- 有多少个类别就输出多少个字典，其中字典的`key`是属性与值（也就是划分），值是相应的概率，例如`survived[age=1]=2/5(age=1代表age处于第一个划分)`
- `cateRate`：输出类别的概率向量`（P（C=k））`

识别部分，我们通过贝叶斯公式用每一个类别在训练步骤得到的字典去计算他在那一个类别的概率，通过贝叶斯公式我们了解到，这个概率为
//贝叶斯公式
所以在识别部分，我们将待识别的记录构造成与`dataMatrix`的记录格式相同的记录并作为输入：
- 计算输入的记录在每一个类别的后验概率（tips：通过在对应属性的`dict`中找到`ai=j`这样的记录获取相应划分的概率）
- 取最大值
期望输出：
- 根据最大值输出相应的类别

至此，识别完成

Todo：将不同划分的概率值通过以`[ai=j]`这样的key储存在字典里效率很低，希望以后寻找到更好的解决办法


# 算法实现
下列算法只是粗糙的基于`Python`实现，使用`Titanic.dat`数据集，用前800条数据进行训练并用后82条数据测试，正确率为81%左右

## 预处理部分
与前面的总结对应，首先进行数据的预处理，首先**读入训练数据，定义需要输出的`dataMatrix`和`cateVec`向量**
```
def preProcess():
    # 通过csv模块读入训练数据
    reader=csv.reader(open("train.csv"))
    cateVec=[]
    dataMatrix=[]

```
**循环train.csv的每一条数据，首先提取需要的属性并将没有值的数据设为默认值**
```
for id,sur,pc,name,sex,age,sib,par,tic,fare,car,emb in reader:
    # 标题行跳过
    if(id=='PassengerId'):
        continue
    # 提取需要数据
    line = [pc, sex, age, sib, par]
    # 给空值填充值
    for i in range(len(line)):
        if (len(line[i])==0):
```
**将数据处理为需要的格式并处理划分**
```
        #转化为需要的格式
        age=float(line[2])
        sib=int(line[3])
        par=int(line[4])
        sur=int(sur)

        # 划分数据
        ageTemp = '0'
        if 0 < age and age < 10:
            ageTemp = '1'
        elif 10 <= age < 20:
            ageTemp = '2'
        elif 20 <= age < 30:
            ageTemp = '3'
        elif 30 <= age < 40:
            ageTemp = '4'
        elif age >= 40:
            ageTemp = '5'

        sibspTemp = '0'
        if sib == 1:
            sibspTemp = '1'
        elif sib == 2:
            sibspTemp = '2'
        elif sib > 2:
            sibspTemp = '3'

        parchTemp = '0'
        if par == 1:
            parchTemp = '1'
        elif par == 2:
            parchTemp = '2'
        elif par > 2:
            parchTemp = '3'
```
**将类别加入类别集合，每一条处理完的数据加入数据矩阵,最后返回**
```
        cateVec.append(sur)
        dataMatrix.append([pc, '1' if sex == 'male' else '0', ageTemp, sibspTemp, parchTemp])
    return dataMatrix,cateVec
```

## 训练部分
下列是训练部分
**首先算出需要输出的类别概率`P（C=0）`**
```
def train(data, cateVec):
    # 类别概率（P（C=0））
    surRate=sum(cateVec)/len(cateVec)
```
**然后循环计算需要输出的每个划分出现的次数**
```
    # 不同类别下不同划分的次数
    surVec={}
    unsurVec={}
    # 循环dataMatrix里的每一个属性，根据不同的类别统计其对应的划分的次数
    for i in range(len(data)):
        for j in range(len(data[i])):# 循环一条记录的每一个属性，存入的字符串形如"0=2"，代表第一个属性在第二个划分区间里的次数
            if(cateVec[i]==0): # 如果未幸存
                if (str(j)+'='+data[i][j]) in unsurVec:
                    unsurVec[(str(j)+'='+data[i][j])]+=1
                else:
                    unsurVec[(str(j)+'=' + data[i][j])] =1
            else:# 如果幸存
                if (str(j)+'='+data[i][j]) in surVec:
                    surVec[(str(j)+'='+data[i][j])]+=1
                else:
                    surVec[(str(j)+'=' + data[i][j])] =1
```
**将次数除以类别出现的次数即为相应的先验概率（P（ai=j|C=k））**
```
# 对于储存的每一个值除以类别的概率即为P（ai=j|C=k）
    for key in unsurVec.keys():
        targetStr = key[key.find('|') + 1:]
        unsurVec[key]/=(len(cateVec) * (1 - surRate))

    for key in surVec.keys():
        targetStr = key[key.find('|') + 1:]
        surVec[key]/=(len(cateVec) * surRate)
    return unsurVec,surVec,surRate
```
## 识别部分
识别部分将输入的向量构造成形如字典存储的"0=2"的形式，代表第0个属性值为2（处于第二个划分），再到不同类别的概率字典中取出值乘以类别的概率，即（P（ai=j|C=k）\*（P（C=k）））为贝叶斯公式的分子，因为分母相同，取分子最大的类别即为结果
```
def classify(feature,unsurVec,surVec,survivedRate):
    # 初始化结果概率
    survivedPos=1
    unsurvivedPos=1
    # 对于每一个属性，计算P(ai=j|C=k)，并将结果乘入对应类别的结果概率
    for i in range(len(feature)):
        tempStr=str(i)+'='+feature[i]
        survivedPos*=surVec[tempStr]
        unsurvivedPos*=unsurVec[tempStr]
    # 计算P(ai=j|C=k)*P（C=k）
    survivedPos*=survivedRate
    unsurvivedPos*=(1-survivedRate)
    # 返回概率大的类别
    return 1 if survivedPos>unsurvivedPos else 0

```
## 主程序调用
```
data,survived=preProcess()
unsurVec,surVec,surRate=train(data,survived)
k=0 # 准确的个数
total=92 # 测试个数

# 对输入数据预处理，得到与dataMatrix相同的数据格式
reader=csv.reader(open("test.csv"))
for id, sur, pc, name, sex, age, sib, par, tic, fare, car, emb in reader:
    if (id == 'PassengerId'):
        continue

    temp = [pc, sex, age, sib, par]
    for i in range(len(temp)):
        if (len(temp[i]) == 0):
            temp[i] = 1

    age = float(temp[2])
    sib = int(temp[3])
    par = int(temp[4])

    ageTemp = '0'
    if 0 < age and age < 10:
        ageTemp = '1'
    elif 10 <= age < 20:
        ageTemp = '2'
    elif 20 <= age < 30:
        ageTemp = '3'
    elif 30 <= age < 40:
        ageTemp = '4'
    elif age >= 40:
        ageTemp = '5'

    sibspTemp = '0'
    if sib == 1:
        sibspTemp = '1'
    elif sib == 2:
        sibspTemp = '2'
    elif sib > 2:
        sibspTemp = '3'

    parchTemp = '0'
    if par == 1:
        parchTemp = '1'
    elif par == 2:
        parchTemp = '2'
    elif par > 2:
        parchTemp = '3'

    temp = [pc, '1' if sex == 'male' else '0', ageTemp, sibspTemp, parchTemp]
    # 判断识别结果
    if(classify(temp,unsurVec,surVec,surRate)==int(sur)):
        k+=1

# 输出识别率        
print(k/total)

```

**[Github项目地址](https://github.com/whatseven/nbm-base.git)**

# Todo
对于先验概率的储存还是不太满意，应该会有更好的数据结构用来存储对于每一个类别，每一个属性的每一个划分的值

#参考资料：
> [算法杂货铺——分类算法之朴素贝叶斯分类(Naive Bayesian classification)](http://www.cnblogs.com/leoo2sk/archive/2010/09/17/naive-bayesian-classifier.html)
