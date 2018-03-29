---
layout: post
title: 农业知识图谱项目展示及说明
category: knowledgeGraph
tag: demo
author: Chen
---


Agriculture_KnowledgeGraph
demo：http://39.108.100.170:8000 
github：https://github.com/qq547276542/Agriculture_KnowledgeGraph 
欢迎star&fork~







###  命名实体识别
进入主页面，输入文本，即可看到以下命名实体和分词的结果（确保django和neo4j都处于开启状态）：

![](https://raw.githubusercontent.com/qq547276542/blog_image/master/agri/2.png)

点击实体的超链接，可以跳转到词条页面：

![](https://raw.githubusercontent.com/qq547276542/blog_image/master/agri/3.png)

### 关系查询
关系查询部分，我们能够搜索出与某一实体相关的实体，以及它们之间的关系： 
![](https://raw.githubusercontent.com/qq547276542/blog_image/master/agri/7.png)

### 知识的树形结构
农业知识概览部分，我们能够列出某一农业分类下的词条列表，这些概念以树形结构组织在一起：

![](https://raw.githubusercontent.com/qq547276542/blog_image/master/agri/6.png)

### 农业分类的树形图：

![](https://raw.githubusercontent.com/qq547276542/blog_image/master/agri/5.png)

### 训练集标注
我们还制作了训练集的手动标注页面，每次会随机的跳出一个未标注过的词条。链接：http://localhost:8000/tagging-get , 手动标注的结果会追加到/label_data/labels.txt文件末尾：

![](https://raw.githubusercontent.com/qq547276542/blog_image/master/agri/4.png)

### 思路
#### 图谱实体获取：
1.根据19000条农业网词条，按照筛法提取名词（分批进行，每2000条1批，每批维护一个不可重集合）

2.将9批词做交集，生成农业词典

3.将词典中的词在互动百科中进行爬取，抛弃不存在的页面，提取页面内容，存到数据库中

4.根据页面内容，提取每一个词条页面的特征，构造相似度的比较方法，使用KNN进行分类

5.最后获取每个词条的所属类别，同时能够剔除不属于农业的无关词条

#### 命名实体识别：
使用thulac工具进行分词，词性标注，命名实体识别（仅人名，地名，机构名） 
为了识别农业领域特定实体，我们需要： 
1. 分词，词性标注，命名实体识别 
2. 以识别为命名实体（person，location，organzation）的，若实体库没有，可以标注出来 
3. 对于非命名实体部分，采用一定的词组合和词性规则，在O(n)时间扫描所有分词，过滤掉不可能为农业实体的部分（例如动词肯定不是农业实体） 
4. 对于剩余词及词组合，匹配知识库中以分好类的实体。如果没有匹配到实体，或者匹配到的实体属于0类（即非实体），则将其过滤掉。 
5. 实体的分类算法见下文。

HudongItem
![](https://raw.githubusercontent.com/qq547276542/blog_image/master/agri/1.png)

#### 页面分类
分类器：KNN算法
* 无需表示成向量，比较相似度即可
* K值通过网格搜索得到
* 定义两个页面的相似度sim(p1,p2)：

* title之间的词向量的余弦相似度(利用fasttext计算的词向量能够避免out of vocabulary)
  2组openType之间的词向量的余弦相似度的平均值

* 相同的baseInfoKey的IDF值之和（因为‘中文名’这种属性贡献应该比较小）
* 相同baseInfoKey下baseInfoValue相同的个数
* 预测一个页面时，由于KNN要将该页面和训练集中所有页面进行比较，因此每次预测的复杂度是O(n)，n为训练集规模。在这个过程中，我们可以统计各个分相似度的IDF值，均值，方差，标准差，然后对4个相似度进行标准化:(x-均值)/方差

* 上面四个部分的相似度的加权和为最终的两个页面的相似度，权值由向量weight控制，通过10折叠交叉验证+网格搜索得到

#### Labels：（命名实体的分类）


| Label	| NE Tags	| Example
| -------| :-----: | :-----: |
|0	|Invalid（不合法）|	“色调”，“文化”，“景观”，“条件”，“A”，“234年”（不是具体的实体，或一些脏数据）|
|1	|Person（人物，职位）|	“袁隆平”，“习近平”，“皇帝”|
|2	|Location（地点，区域）|	“福建省”，“三明市”，“大明湖”
|3	|Organization（机构，会议）|	“华东师范大学”，“上海市农业委员会”|
|4	|Political economy（政治经济名词）|	“惠农补贴”，“基本建设投资”
|5	|Animal（动物学名词，包括畜牧类，爬行类，鸟类，鱼类，等）|	“绵羊”，“淡水鱼”，“麻雀”|
|6	|Plant（植物学名词，包括水果，蔬菜，谷物，草药，菌类，植物器官，其他植物）|	“苹果”，“小麦”，“生菜”|
|7	|Chemicals（化学名词，包括肥料，农药，杀菌剂，其它化学品，术语等）|	“氮”，“氮肥”，“硝酸盐”，“吸湿剂”|
|8	|Climate（气候，季节）|	“夏天”，“干旱”|
|9	|Food items（动植物产品）|	“奶酪”，“牛奶”，“羊毛”，“面粉”|
|10	| Diseases（动植物疾病）	| “褐腐病”，“晚疫病”|
|11	|Natural Disaster（自然灾害）	“地震”，“洪水”，“饥荒”|
|12	|Nutrients（营养素，包括脂肪，矿物质，维生素，碳水化合物等）|	“维生素A”，”钙”|
|13	|Biochemistry（生物学名词，包括基因相关，人体部位，组织器官，细胞，细菌，术语）	“染色体”，“血红蛋白”，“肾脏”，“大肠杆菌”|
|14	|Agricultural implements（农机具，一般指机械或物理设施）|	“收割机”，“渔网”|
|15	|Technology(农业相关术语，技术和措施)	|“延后栽培”，“卫生防疫”，“扦插”|
|16	|other（除上面类别之外的其它名词实体，可以与农业无关但必须是实体）|	“加速度”，“cpu”，“计算机”，“爱鸟周”，“人民币”，“《本草纲目》”，“花岗岩”|

