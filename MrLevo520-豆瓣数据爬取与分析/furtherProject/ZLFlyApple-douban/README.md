- Python  3.6
- Jupyter Notebook 5.0
- ECharts

----------

起因
--
> DT君Python精英群第9次作业，使用selenium爬取豆瓣电影-冷门佳片Top400（按热度排序）的豆瓣评分及相关Tag（文件 douban400MovieDetail）。

----------

目的
--
1. 对冷门佳片按Tag进行分组统计
2. 使用ECharts进行结果展示
3. 学习Pandas的基本使用

----------

方案
--
>  使用 Jupyter Notebook + Pandas + ECharts 为开发、统计、结果展示组合。

----------

实现过程
----
1. 将爬取的电影数据按格式存入Pandas 的DataFrame中。
2. 对感兴趣的方向、内容进行统计。
3. 根据ECharts相关图表的数据存储结构进行统计结果输出、展示。

> 以豆瓣高分，然后按评分排序的点击过程（其余操作一致，先种类后排序选择，再爬）

![这里写图片描述](http://img.blog.csdn.net/20160719165615514)

----------


实现代码
----

```python
# -*- coding: utf-8 -*-
#Author:FlyApple
#分析豆瓣电影冷门 Top400

import re
import pandas as pd
####################################################
#读取原始文件，分别存储list
####################################################
movies = []
movie  = []
sum = 0 #记录读取电影数量
with open('C:/Users/FlyApple/douban400MovieDetail',encoding = 'ansi') as f:
    for line in f:
        #找到数据头，开始处理
        if '-----NO' in line:
            movie.clear()
            sum = sum +1
            count = 0 #对原始数据文件按行计数
            point = 0 #处理“导演”、“语言”等项目的数据缺失问题
            no = int(re.findall("\d+",line)[0])
            movie.append(int(no))
            ml = list(movie) #没有这句，movies中的元素都相同
            print(ml,movie)
            movies.append(ml)
        else:
            if count == 0:
                name = (line.split(' ')[0]).strip()
                ml.append(name)
            elif count ==1:
                year = int(line.split('：')[-1])
                ml.append(year)
            elif count ==3:
                score = float(line)
                ml.append(score)
            elif count ==4:
                people = int(re.findall("\d+",line)[0])
                ml.append(people)
            elif "导演" in line:
                dire = (line.split(': ')[-1]).strip()
                point += 1
                ml.append(dire) 
            elif "类型" in line:
                dire = (line.split(': ')[-1]).strip()
                if point<1:
                    point+=1
                    ml.append('')
                point += 1
                ml.append(dire)
            elif "国家" in line:
                dire = (line.split(': ')[-1]).strip()
                if point <1:
                    point+=1
                    ml.append('')      
                if point <2:
                    point+=1
                    ml.append('')
                point += 1
                ml.append(dire)
            elif "语言" in line:
                dire = (line.split(': ')[-1]).strip()
                if point <1:
                    point+=1
                    ml.append('')
                if point <2:
                    point+=1
                    ml.append('')
                if point <3:
                    point+=1
                    ml.append('')
                point += 1
                ml.append(dire)
            elif "片长" in line:
                length = int(re.findall("\d+",line)[0])
                if point <1:
                    point+=1
                    ml.append('')                
                if point <2:
                    point+=1
                    ml.append('')
                if point <3:
                    point+=1
                    ml.append('')
                if point <4:
                    point+=1
                    ml.append('')
                point += 1
                ml.append(length)
            count += 1
####################################################
#将将list格式数据转为pandas.DataFrame数据并处理
####################################################
data1 = pd.DataFrame(movies, columns=['排名', '片名', '上映年份', '豆瓣评分','打分人数','导演','类型','国家','语言','片长'])

####################################################
#统计处理1：
#按“上映年份”等Tag简单计数
####################################################

kkk = data1.groupby(['上映年份']).size()
#以下按ECharts条形图数据格式输出
a = ((kkk.values))
b = ((kkk.index))
for kk in a:
    print(str(kk)+",")
for kk in b:
    print("'"+str(kk)+"',")

    
####################################################
#统计处理2：
#按“上映年份”等按区间分组计数（桶分析）
####################################################

#按year划分区间，含左不含右，例如[1920,1930)
year = [1920,1930,1940,1950,1960,1970,1980,1990,2000,2010,2020]
yearcats = pd.cut(data1.loc[:,'上映年份'],year,right=False)
grouped = data1.groupby(yearcats)
kkk=grouped.describe()

a = ((kkk.values))
b = ((kkk.index))
for kk in a:
    print(str(kk[25])+",")
for kk in b:
    print("'"+str(kk)+"',")

    
####################################################
#统计处理3：
#统计1980-2010期间电影的评分Top15
#################################################### 

data2 = data1[(data1['上映年份'] > 1979) & (data1['上映年份'] < 2011)]
aa=data2.sort_values(by = '豆瓣评分',ascending=False)#按评分进行排序
aa[:15]


####################################################
#统计处理4：
#对于形如 喜剧 / 动画 / 奇幻 属于多个分类的项目，
#首先进行元素拆分并分别统计
####################################################

data2 = data1
Name = "导演"
genre_iter = (set(k.strip() for k in x.split("/")) for x in data2.loc[:,Name])
genres = sorted(set.union(*genre_iter))

dummies = pd.DataFrame(np.zeros((len(data2),len(genres))),columns=genres)
for i,gen in enumerate(data2.loc[:,Name]):
    for k in gen.split("/"):
        dummies.ix[i,k.strip()]=1 #包含则记为1
        
movies_windic = data2.join(dummies)

result = movies_windic.sum()[10:].sort_values(ascending=False)
#将结果按ECharts表格所需格式输出
```

----------


实现效果
----
![DTTest](https://raw.githubusercontent.com/ZLFlyApple/DTTest/master/%E5%AF%BC%E6%BC%94%E4%BD%9C%E5%93%81%E5%9B%BE.png)

![](https://raw.githubusercontent.com/ZLFlyApple/DTTest/master/%E5%86%B7%E9%97%A8%E4%BD%B3%E7%89%87top400%E5%9B%BD%E5%AE%B6%E5%88%86%E5%B8%83.png)


![DTTest](https://raw.githubusercontent.com/ZLFlyApple/DTTest/master/%E5%86%B7%E9%97%A8%E4%BD%B3%E7%89%87top400%E5%B9%B4%E4%BB%A3%E5%88%86%E5%B8%83.png)

![DTTest](https://raw.githubusercontent.com/ZLFlyApple/DTTest/master/%E5%86%B7%E9%97%A8%E4%BD%B3%E7%89%87top400%E8%AF%AD%E8%A8%80%E5%88%86%E5%B8%83.png)

![1980-2010 Top15](https://raw.githubusercontent.com/ZLFlyApple/DTTest/master/1980-2010top15.PNG)



----------


问题及解决方案
----

Q: movie中的更新与movies绑定？

A:  列表嵌套还不太熟悉，目前的解决方法为设置ml = List(movie)，movies.append(ml)。

----------

参考
----
- 统计处理3、4部分主要参照《利用Python进行数据分析》相关部分。

