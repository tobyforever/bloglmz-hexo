---
title: elasticsearch数据库笔记
date: 2023-09-01 20:43:51
tags: 数据库
toc: false
published: true
---
# 知识点

#### 索引的属性配置：
静态配置创建后不能改，动态配置创建后可以改。
静态配置：
index.number_of_shards分片数
index.routing_partition_size
动态配置：
index.number_of_replicas副本数，默认为1
index.refresh_interval最新数据对搜索可见的刷新间隔，默认1s（ES是近实时的而非全实时的！）

#### ES的字段类型映射
动态映射和静态映射 映射相当于关系数据库的字段类型信息，在索引的mappings字段里。
静态映射是创建索引的时候手动指定好字段的类型
动态映射是直接创建索引并写入文档，ES自动推测字段类型。
如果对入库时要求限制字段类型，就要设置mapping.dynamic为strict抛出异常或者false忽略新字段，默认为true自动推测类型添加新字段

#### es的字段类型
文本不叫做string，而是分位text和keyword，text是全文检索的（会分词）text 类型的字段不用于排序，且很少用于聚合（Terms Aggregation 除外），keyword是用于精确值搜索的，不做分词

字段的index的选项可以设置analyzed（默认），not_analyzed,no
- analyzed 所有字母转小写然后分词 效果是模糊匹配
- not_analyzed是不分词，整个字符串精准匹配
- no这个字段不参与索引（不用于检索），可以节省索引的存储空间。

数字类型注意根据取值范围选择合适的类型，[字段的长度越短，索引和搜索的效率越高](https://www.knowledgedict.com/tutorial/elasticsearch-index-mapping.html)

处理浮点数时，优先考虑使用 scaled_float 类型(底层是整数的小数，适用于价格、有效数字位数固定的地理经纬度)，因为压缩整数比浮点数更节省存储空间。

日期类型，输入可以是格式化的字符串、毫秒时间戳、秒级时间戳； ES内部会转化为UTC用ms级长整型时间戳保存，因为整数存储和处理更快。

布尔型只接受true,false, "true","false"

Elasticsearch 没有专用的数组类型，默认情况下任何字段都可以包含 0 个或者多个值，但是一个数组中的值必须是同一种类型. 动态添加数据时，数组的第一个值的类型决定整个数组的类型. 
数组类型不需要提前配置，直接写入数据即可，并且可以搜索例如按lists.name搜索lists字段的name

nested类型 嵌套对象类型 （索引一个包含 100 个 nested 字段的文档实际上就是索引 101 个文档，每个嵌套文档都作为一个独立文档来索引）

地理类型：
geo_point
geo_shape
geo point 类型用于存储地理位置信息的经纬度，可用于以下几种场景：

- 查找一定范围内的地理位置。
- 通过地理位置或者相对中心点的距离来聚合文档。
- 把距离因素整合到文档的评分中。
- 通过距离对文档排序。
geo_point的输入格式有：
- lat和lon的经纬度键值对
- 字符串格式的逗号分隔经纬度（纬度在前）
- geohash
- 数组 按格式[经度小数,纬度小数]

geo_shape类型可以存储[GeoJSON格式的平面几何类型](https://www.knowledgedict.com/tutorial/elasticsearch-index-mapping.html)

ES的特殊数据类型包括ip、completion范围、token_count令牌计数、attachment附件、percolator抽取


#### ElasticsearchRestTemplate使用
https://blog.csdn.net/csdn_avatar_2019/article/details/128976578


# 技巧

如果索引比较大，需要定期重建，则可以用这个平滑滚动切换技巧
[使用索引别名（链接）来对索引进行引用和平滑切换](https://www.knowledgedict.com/tutorial/elasticsearch-index-smooth-shift.html)


Elasticsearch 可以不用这样，它提供了两个维度的拆分方式，第一维度采用多个索引命名拆分，第二维度采用索引多分片，对于查询来说，可以灵活匹配索引，一次指定一个索引，也可以一次指定多个索引

https://zhuanlan.zhihu.com/p/145021757

ES索引拆分方案
https://elasticsearch.cn/article/14113#tip0

