# 数据说明  

数据总体包括：用户基本信息表、银行流水表、浏览记录表、信用账单表、放款时间表和违约记录表。 

用户信息表字段：用户id，性别，职业，学历，婚姻，户口类型  
银行流水表字段：用户id，交易时间，交易类型，交易金额，工资标签  
浏览记录表字段：用户id，浏览时间，浏览行为，浏览子行为编号  
放款表字段：用户id，放款时间  
违约记录表字段：用户id，违约标签  
账单表字段：用户id，账单时间，银行id，上期账单额，上期还款额，信用额度，当期账单余额，当期最  
低还款额，消费次数，当期账单额，调整额度，循环利息，可用余额，预借现金额度，还款状态  

特殊说明：用户信息表中性别为0表示未知。涉及时间戳字段为0表示未知。违约记录中0表示不违约，1表示  
违约。银行流水表，交易类型，1表示支出，0表示收入，工资收入为1。  




# 目标  

利用上述数据，构建违约预测模型，以用来预测客户是否会出现逾期  


# 流程  

  * 数据初步处理  
  * 数据基本清洗  
  * 特征构建  
  * 异常值处理  
  * 特征相关性探索  
  * 建模  



# 数据模型构建  


## 初步探索  
导入数据，检查数据是否有缺失。对每个表都进行此步操作。  
![图1](https://github.com/ElyCman/CreditPredict/blob/main/pictures/1.png)  
银行和账单，这两个表的时间有缺失，但是账单的缺失比率占表的20%  
![图2](https://github.com/ElyCman/CreditPredict/blob/main/pictures/2.png)  
占比过大，因此不删除  
银行、账单和浏览表，这三个表有记录重复情况，予以删除重复记录  
![图3](https://github.com/ElyCman/CreditPredict/blob/main/pictures/3.png)  
检查重复记录缺失情况后，对几个表的用户数进行统计，可以看到如下结果，  
![图4](https://github.com/ElyCman/CreditPredict/blob/main/pictures/4.png)  
即，用户表的用户数： 55596， 银行流水表的用户数： 9294， 账单表的用户数： 53174， 浏览表  
用户数： 47330，放款表用户数： 55596， 逾期表用户数： 55596   


## 特征构建  

用放款表的时间戳对银行、浏览和账单的表进行筛选，留下小于放款时间的用户信息。譬如  
![图5](https://github.com/ElyCman/CreditPredict/blob/main/pictures/5.png)  
此时留下的银行流水中用户的个数9268个。其他表做相同操作。  

### 构建银行特征  
![图6](https://github.com/ElyCman/CreditPredict/blob/main/pictures/6.png)  
收入：  
![图7](https://github.com/ElyCman/CreditPredict/blob/main/pictures/7.png)   
构建好该指标后，看看其分布情况  
![图8](https://github.com/ElyCman/CreditPredict/blob/main/pictures/8.png)   
符合正态分布情况，没有出现偏态。但是有异常值。  
支出：  
以同样方法构建支出指标，器分布如下  
![图9](https://github.com/ElyCman/CreditPredict/blob/main/pictures/9.png)   
工资收入：  
以相同方法构建工资收入特征， 但是该特征出现右偏  
![图10](https://github.com/ElyCman/CreditPredict/blob/main/pictures/10.png)   
对该指标进行对数化处理  

### 相关性  
三个构建的特征的彼此相关性如何？用seaborn的热力图进行展示  
![图11](https://github.com/ElyCman/CreditPredict/blob/main/pictures/11.png)
明显看到，收入和支出存在极大相关，我们预留一个特征。待所有特征构建完成后，进行iv处理，看看留  
下哪个特征比较好。  

### 浏览记录  
浏览行为数据则出现左偏，用对数处理  
![图12](https://github.com/ElyCman/CreditPredict/blob/main/pictures/12.png)  

### 账单特征  

第一个，可以先构建银行卡数量指标，再构建其他指标。新增四个衍生变量：  
上期未还账单额=上期账单额-上期还款额  
相邻两期账单金额差=本期账单金额-上期账单金额  
本期还款总额=上期账单金额-上期还款金额+本期账单金额-调整金额+循环利息  
已使用信用额度=信用额度-可用余额  
此时，账单表的字段就有17个  

### 相关性  

相关性分析表明，上期账单金额与上期还款额相关系数为0.78，当期还款总额与上期账单金额的相关系数为0.7，  
当期账单额与上期还款额相关系数为0.65，当期最低还款额与当期账单余额有很大相关性，系数达到了0.83，  
所构建的新特征已使用信用额度与信用额度相关性系数为0.94  
![图13](https://github.com/ElyCman/CreditPredict/blob/main/pictures/13.png)    


## 特征选择  
把处理好的银行、浏览记录、账单、用户信息、放款和违约记录表合成一个表，最终得到的用户数5730  
![图14](https://github.com/ElyCman/CreditPredict/blob/main/pictures/14.png)   
合并后，对特征进行重要性对比  
![图15](https://github.com/ElyCman/CreditPredict/blob/main/pictures/15.png)  
结合前面的分析，去掉的变量：支出、上期账单金额，当期账单余额，当期账单金额，已用信用额，可用余额，调  
整额度，户口，学历，婚姻等变量。变量筛选后，就可以建立模型了。  


## 建模  

### 数据分割  
![图16](https://github.com/ElyCman/CreditPredict/blob/main/pictures/16.png)    
把数据分为训练集和测试集，用于训练模型和检验模型  

### 训练模型  
决策树。用网格搜索，返回最佳模型参数。  
![图17](https://github.com/ElyCman/CreditPredict/blob/main/pictures/17.png)   
决策树的准确率约0.8581，依此方法，随机森林的准确率约为0.8551，逻辑回归则约为0.8575  




# 总结  
* 第一、对银行业务的不了解限制了特征工程的工作，这对模型的提升是很不利的；  
* 第二、模型参数需要继续探究，以此优化模型；  
* 第三、本地机器限制了支持向量机等其他模型的使用  
