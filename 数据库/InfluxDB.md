# InfluxDB
## InfluxDB概要
InfluxDB是一种时序数据库，数据按照时间顺序存储，对于日志、时序的数据的查询存储是一个很好的选择。
## InfluxDB的数据格式
InfluxDB中存储的数据被称为时间序列数据，时序数据由多个**数据点**组成，数据点又由**time**（时间戳）、**measurement**（测量，统计）、至少一个**k-v格式的field**和**零个或多个tag**组成。  
在概念上可以将measurement理解为mysql中的table，其主键总是time，tag和field是除主键外的其他列，区别是tag是被索引的，而field不是。在InfluxDB中，你可以由数百万个measurement，而无需事先定义其scheme，并且null不会被存储。
* 插入数据：需要以数据点的形式进行插入。
```
insert <measurement>[,<tag-key>=<tag-value>...] <field-key>=<field-value>[,<field2-key>=<field2-value>...] [unix-nano-timestamp]
```
中扩号中的内容，是可选的，可以有，有可以没有。例如：  
```
insert cpu,host=serverA,region=us_west value=0.64
payment,device=mobile,product=Notepad,method=credit billed=33,licenses=3i 1434067467100293230
```
time可以被省略，则取默认时间  
* 查询数据：   
```
SELECT "host", "region", "value" FROM "cpu"
```
注意不区分tag和field。若要查询全部field和tag，与mysql一样使用通配符\*。 至于其他查询语法，看参考文献或官方文档  
* InfluxDB可以直接通过HTTP接口来创建数据库和读写数据，默认端口8086。
## 数据采样和保留
InfluxDB若要存储很长时间的数据，可以对数据进行采样，对于高精度的数据和低精度的数据设置不同的保存策略。  
#### 连续查询  Continuous Query （CQ）
  连续查询是在InfluxDB中周期性运行的一个InfluxQL查询，其目的是利用`SELECT`函数添加一个`GROUP BY time()`来对数据进行采样
#### 保留策略  Retention Policy  (RP)
  保留策略是InfluxDB数据架构的一部分，它描述了InfluxDB保存数据的时间。InfluxDB会比较服务器的本地时间和请求数据中的时间戳，删除比你在RPs中使用`DURATION`设置的更老的数据，默认为永久保存。一个数据库中可以为不同数据指定不同RP，但数据库的默认RP只有一个。
#### 数据采样和保留
数据的采样和保留就是结合CQ和RP来实现的，利用CQ可以实现数据降准，将细粒度数据转为粗粒度数据；利用RP可以减小存储空间，保留有价值的数据。  
结合CQ和RP可以对不同粒度的数据采取不同的保留策略，比如细粒度的保存时间短，用来检索短期数值，粗粒度的保存时间长，用来反映长期趋势。  
## tag和field的区别
tag包含索引，适合存储元数据（又称中介数据、中继数据，为描述数据的数据（data about data），主要是描述数据属性（property）的信息，用来支持如指示存储位置、历史数据、资源查找、文件记录等功能。可以简单理解为**目录**）。我们可以设计tag作为查询条件，查询速度会比field快，但是tag过多的话也会增大数据插入的时间，所以tag也不能冗余。
**InfluxDB中可以没有tag，但不能没有field。另外，不要使用field作为查询条件**
## 序列series
series是RP，measurement和tag set的集合，是数据源的合的概念。在**同一个database中，相同retention policy、相同measurement、相同tag的数据属于一个series集合**，标识这条数据来自哪里，同一个series的数据在物理上按照时间顺序排列在一起。  
通过`show series;`可以查询到上述条件的数据。

## 指针/数据点 Points
InfluxDB中的数据由时间序列化结构构成，时间序列化结构包括0个到多个指针(points)，每个指针都是**一个离散的指标中实际的取样值**，指针(points)由timestamp、measurement、0个到多个tag、大于等于一个的field字段等共同构成，类似RDBMS中的row的概念。
## 参考文献
https://www.hellodemos.com/hello-influxdb/influxdb-influxdb-intro.html  
https://www.cnblogs.com/ilifeilong/p/12745565.html  
https://www.influxdata.com/  
https://www.cnblogs.com/owenzh/p/13826805.html  
https://www.jianshu.com/p/b499347ea73a  