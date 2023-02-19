---
title: Elasticsearch教程
tags: 常用框架
categories: [常用框架, elasticSearch]
cover: https://gitee.com/studentgitee/note-picture/raw/master/c38cf6948b39dd755ac13fc639070fd8af5cd4db.png
---

![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

# Elasticsearch教程(一)

## 一. 前序

> Elasticsearch是一个基于Apache Lucene(TM)的开源搜索引擎。无论在开源还是专有领域，Lucene可以被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库。
>
> 但是，Lucene只是一个库。想要 使用它，你必须使用Java来作为开发语言并将其直接集成到你的应用中，更糟糕的是，Lucene非常复杂，你需要深入了解检索的相关知识来理解它是如何工作的。
>
> Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的 RESTful API 来隐藏Lucene的复杂性，从而让全文搜索变得简单。
>
> Elasticsearch的中文网址：https://www.elastic.co/cn/products/elasticsearch

### 正向索引和倒排索引

> 正向索引与倒排索引，这是在搜索领域中非常重要的两个名词，正向索引通常用于数据库中，在搜索引擎领域使用的最多的就是倒排索引，我们根据如下两个网页来对这两个概念进行阐述：

**html1**

```java
我爱我的祖国，我爱编程
```

**html2**

```java
我爱编程，我是个快乐的小码农
```

#### 正向索引

> 假设我们使用mysql的全文检索，会对如上两句话分别进行分词处理，那么预计得到的结果如下：

```java
我 爱 爱我 祖国 我的祖国 编程 爱编程 我爱编程
```

```java
我 我爱 爱 编程 爱编程 我爱编程 快乐 码农 小码农
```

> 假设我们现在使用正向索引搜索 **中国** 这个词，那么会到第一句话中去查找是否包含有 **中国** 这个关键词，如果有则加入到结果集中；第二句话也是如此。假设现在有成千上百个网页，每个网页非常非常的分词，那么搜索的效率将会非常非常低些。

#### 倒排索引

> 倒排索引是按照分词与文档进行映射，我们来看看如果按照倒排索引的效果：

![image-20230215153918121](https://gitee.com/studentgitee/note-picture/raw/master/image-20230215153918121.png)

> 如果采用倒排索引的方式搜索**祖国** 这个词，那么会直接找到关键词中查找到 **祖国** ，然后查找到对应的文档，这就是所谓的倒排索引。

## 二. 软件简介以及启动

### 相关软件下载地址

|    软件名     |                   下载地址                   |
| :-----------: | :------------------------------------------: |
| Elasticsearch |       https://www.elastic.co/cn/start        |
|    Kibana     |       https://www.elastic.co/cn/start        |
|   Logstash    | https://www.elastic.co/cn/downloads/logstash |
|     canal     |  https://github.com/alibaba/canal/releases   |

### Elasticsearch安装

> 进入到 elasticsearch 解压目录下的 bin 目录下，双击 elasticsearch.bat 即可启动。
> 在浏览器地址栏输入: http://localhost:9200/ ，如果出现如下页面表示 elasticsearch 启动成功

![image-20230215154119739](https://gitee.com/studentgitee/note-picture/raw/master/image-20230215154119739.png)

### Kibana

#### Kibana简介

> Kibana是世界上最受欢迎的开源日志分析平台ELK Stack中的“K” ，它为用户提供了一个工具，用于在存储于Elasticsearch集群中的日志数据进行检索，可视化和构建仪表板。
>
> Kibana的核心功能是数据查询和分析。使用各种方法，用户可以搜索Elasticsearch中索引的数据，以查找其数据中的特定事件或字符串，以进行根本原因分析和诊断。基于这些查询，用户可以使用Kibana的可视化功能，允许用户使用图表，表格，地理图和其他类型的可视化以各种不同的方式可视化数据。

#### Kibana的启动

> 进入到 kibana 解压目录下的 bin 目录下，双击 kibana.bat 即可启动 kibana.
> 在浏览器地址栏输入：http://localhost:5601，出现如下页面代表 kibana 启动成功。

![image-20230215154209405](https://gitee.com/studentgitee/note-picture/raw/master/image-20230215154209405.png)

### Logstash

#### logstash简介

> Logstash是一个开源的服务器端数据处理管道，可以同时从多个数据源获取数据，并对其进行转换，然后将其发送到你最喜欢的“存储”。创建于2009年，于2013年被elasticsearch收购

![image-20230215154235736](https://gitee.com/studentgitee/note-picture/raw/master/image-20230215154235736.png)

#### logstash导入数据

> 虽然 kibana 提供了一些数据集供我们使用，为了加深对 logstash 的理解，我们 movielens的电影数据集。
>
> movielens 数据集的下载地址为：http://files.grouplens.org/datasets/movielens，进入该网页只用下载 ml-latest.zip 数据即可，如下图所示：

![image-20230215154306939](https://gitee.com/studentgitee/note-picture/raw/master/image-20230215154306939.png)

> 将 ml-latest.zip 加压文件中的 movies.csv 文件拷贝到 logstash 的家目录下；
> 再将 logstash 的 config 目录下新建名为 logstash.conf 的文件，文件内容如下:

```java
input {
file {
 # 引号的的内容为 movies.csv 的实际路径，根据实际情况而定
 path => "D:/logstash-datas/movies.csv"
 start_position => "beginning"
 sincedb_path => "D:/logstash-datas/db_path.log"
}
}
filter {
csv {
 separator => ","
 columns => ["id","content","genre"]
}
mutate {
 split => { "genre" => "|" }
 remove_field => ["path", "host","@timestamp","message"]
}
mutate {
 split => ["content", "("]
 add_field => { "title" => "%{[content][0]}"}
 add_field => { "year" => "%{[content][1]}"}
logstash导入数据的命令
验证 movielens 数据集是否成功导入
打开dos命令行，进入到 logstash 的 bin 目录下，执行如下命令导入 movielens 的数据集。
2.4.3 验证
进入到 kibana 的命令行页面，执行 GET _cat/indices 验证数据是否成功导入
三. Elasticsearch的基本概念
}
mutate {
 convert => {
  "year" => "integer"
 }
 strip => ["title"]
 remove_field => ["path", "host","@timestamp","message","content"]
}
}
output {
 elasticsearch {
  # 双引号中的内容为ES的地址，视实际情况而定
  hosts => "http://localhost:9200"
  index => "movies"
  document_id => "%{id}"
 }
stdout {}
}
```

> 打开dos命令行，进入到 logstash 的 bin 目录下，执行如下命令导入 movielens 的数据集。

> logstash.bat -f D:\logstash-datas\config\logstash.conf

![image-20230215154357817](https://gitee.com/studentgitee/note-picture/raw/master/image-20230215154357817.png)

#### 验证

> 进入到 kibana 的命令行页面，执行 GET _cat/indices 验证数据是否成功导入

![image-20230215154424891](https://gitee.com/studentgitee/note-picture/raw/master/image-20230215154424891.png)

## 三. Elasticsearch的基本概念

### 索引Index

> Elasticsearch中的索引有多层的意思：
> a. 某一类文档的集合就构成了一个索引，类比到数据库就是一个数据库(或者数据库表);
> b.它还描述了一个动作，就是将某个文档保存在elasticsearch的过程也叫索引;
> c. 倒排索引。

### 文档Document

```
具体的一条数据，类比到数据库就是一条记录
```

![image-20230215154515755](https://gitee.com/studentgitee/note-picture/raw/master/image-20230215154515755.png)

### 映射mapping

```
mapping 是ES每一个文档的约束信息，例如属性的类型，是否能被索引等。
```

### DSL

> DSL 是 ES 的查询语言。

### 类比

> 我们通过大家比较熟悉的 Mysql与 ES 的基本概念进行类比

|  Mysql   |           Elasticsearch            |
| :------: | :--------------------------------: |
| database |               Index                |
|  table   |  type(在7.0之后type为固定值_doc)   |
|   Row    |              Document              |
|  Column  |               Field                |
|  Schema  |              Mapping               |
|   SQL    | DSL(Descriptor Structure Language) |
|          |                                    |

> 在7.0之前，一个Index可以创建多个类型，从7.0开始，一个索引只能创建一个类型，也就是 _doc

![146a779da01f53e7f7a8d53132d3c7cf](https://gitee.com/studentgitee/note-picture/raw/master/146a779da01f53e7f7a8d53132d3c7cf.png)

## 四. RestAPI

### 基本CRUD

```java
# 查询file_data索引的数据
GET file_data/_search 
    
# 查询file_data的总数
GET file_data/_count 
    
# 查看所有的索引
GET _cat/indices 
 
# 查看file_data的索引结构
GET /file_data/_mapping
   
# 查询id为24的数据
GET file_data/_doc/24 

# 添加id为1的文档 ，如果没有指定id，ES会自动生成 { "firstname": "will","lastname": "smith" }
POST file_data/_doc/1 

# 创建id为2的文档，如果索引中已存在相同id，会报错； { "firstname": "will","lastname": "smith" }
POST file_data/_create/2 

# 在id位2的文档中添加一个age属性，修改结构 { "doc": { "age": 30 } }    
POST file_data/_update/2
    
# 删除id为2的文档   
DELETE file_data/_doc/2 

# 删除 file_data 索引      
DELETE file_data 

# 创建或者修改文档 { "firstname": "Jack", "lastname": "ma" }
PUT file_data/_doc/1 
    
# 创建file_data索引
PUT /file_data
{
  "mappings": {
    "properties": {
      "id":{
        "type": "keyword"
      },
      "name":{
        "type": "text",
        "analyzer": "ik_max_word",
        "fields": {
          "suggest": {
            "type": "completion",
            "analyzer": "ik_max_word"
          }
        }
      },
      "format":{
        "type": "keyword"
      },
      "type_id":{
        "type": "text",
        "index": false
      },
      "address":{
        "type": "text",
        "index": false
      },
      "attachment": {
        "properties": {
          "content":{
            "type": "text",
            "analyzer": "ik_max_word"
          }
        }
      }
    }
  }
}

```

```
# 查看ik_max_word分词的效果
GET _analyze
{
  "analyzer":"ik_max_word",
  "text": ["child"]
}
```



## 五. Analysis

> analysis(只是一个概念)，文本分析是将全文本转换为一系列单词的过程，也叫分词。analysis是通过analyzer(分词器)来实现的，可以使用Elasticsearch内置的分词器，也可以自己去定制一些分词器。 除了在数据写入的时候进行分词处理，那么在查询的时候也可以使用分析器对查询语句进行分词。
>
> anaylzer是由三部分组成，例如有 Hello a World, the world is beautifu
>
> 2. Character Filter: 将文本中html标签剔除掉。
> 2. Tokenizer: 按照规则进行分词，在英文中按照空格分词。
> 3. Token Filter: 去掉stop world(停顿词，a, an, the, is, are等)，然后转换小写

![image-20230215155046204](https://gitee.com/studentgitee/note-picture/raw/master/image-20230215155046204.png)

### 内置分词器

|     分词器名称      |               处理过程               |
| :-----------------: | :----------------------------------: |
|  Standard Analyzer  |   默认的分词器，按词切分，小写处理   |
|   Simple Analyzer   | 按照非字母切分(符号被过滤)，小写处理 |
|    Stop Analyzer    |  小写处理，停用词过滤(the, a, this)  |
| Whitespace Analyzer |        按照空格切分，不转小写        |
|  Keyword Analyzer   |      不分词，直接将输入当做输出      |
|  Pattern Analyzer   | 正则表达式，默认是\W+(非字符串分隔)  |
|                     |                                      |

### 内置分词器示例

#### Standard Analyzer

```java
GET _analyze
{
 "analyzer": "standard",
 "text": "2 Running quick brown-foxes leap over lazy dog in the summer evening"
}
```

#### Simple Analyze

```JAVA
GET _analyze
{
 "analyzer": "simple",
 "text": "2 Running quick brown-foxes leap over lazy dog in the summer evening"
}
```

#### Stop Analyzer

```JAVA
GET _analyze
{
 "analyzer": "stop",
 "text": "2 Running quick brown-foxes leap over lazy dog in the summer evening"
}
```

#### Whitespace Analyzer

```JAVA
GET _analyze
{
 "analyzer": "whitespace",
 "text": "2 Running quick brown-foxes leap over lazy dog in the summer evening"
}
```

#### Keyword Analyzer

```java
GET _analyze
{
 "analyzer": "keyword",
 "text": "2 Running quick brown-foxes leap over lazy dog in the summer evening"
}
```

#### Pattern Analyzer

```java
GET _analyze
{
 "analyzer": "pattern",
 "text": "2 Running quick brown-foxes leap over lazy dog in the summer evening"
}
```

### 扩展分词器

#### ik分词器

**下载**

> 下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases

**安装**

> IK分词器在任何操作系统下安装步骤均一样: 在ES的home目录下的 plugins 目录下创建名为 ik 的文件夹，然后将下载后的 zip 包拷贝到 ik 解压即可

> K分词器提供了两种分词方式：

| 分词器器名称 |                             说明                             |
| :----------: | :----------------------------------------------------------: |
|   ik_smart   | 会做最粗粒度的拆分，比如会将“中华人民共和国歌”拆分为“中华人民共和国,国歌”，适合 Phrase 查询 |
| ik_max_word  | 会将文本做最细粒度的拆分，比如会将“中华⼈人⺠民共和国国歌”拆分为“中华⼈人⺠民共和国,中华⼈人⺠民,中华,华人,人⺠民共和国,人⺠民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合，适合 Term Query； |

**自定义词库**

> 在很多的时候，业务上的⼀一些词库极有可能不在IK分词器器的词库中，需要去定制属于我们自己的词库。例如下面的例子中， 中国人民 、 up主 被切分为一个个的字，我们希望这两个词语是不被拆分；另外 作为中文的停顿词，也不希望出现在分词中，所以我们需要自定义词库和停顿词库。

#### pinyin分词器

**下载**

> 下载地址: https://github.com/medcl/elasticsearch-analysis-pinyin/releases

**安装**

> pinyin 分词器器在任何操作系统下安装步骤均⼀一样: 在ES的家目录下的 plugins 目录下创建名为pinyin 的⽂文件夹，然后将下载后的 zip 包拷⻉贝到 pinyin 解压即可

# Elasticsearch教程(二)

## 一. Request Body深入搜索

### term查询

> term是表达语义的最小单位，在搜索的时候基本都要使用到term。
>
> term查询的种类有：Term Query、Range Query等。
>
> 在ES中，Term查询不会对输入进行分词处理，将输入作为一个整体，在倒排索引中查找准确的词项。我们也可以使用Constant Score 将查询转换为一个filter，避免算分，利用缓存，提高查询的效率。

#### term与terms

> 查询电影名字中包含有 beautiful 这个单词的所有的电影，用于查询的单词不会进行分词的处理

```java
GET movies/_search
{ 
	"query": {  
		"term": {   
			"title": {    
				"value": "beautiful"  
			} 
		}
	}
}
```

> 查询电影名字中包含有 beautiful 或者 mind 这两个单词的所有的电影，用于查询的单词不会进行分词的处理

```java
GET movies/_search
{ 
	"query": {  
		"terms": {   
			"title": ["beautiful","mind"] 
		}
	}
}
```

#### range

> 查询上映在2016到2018年的所有的电影，再根据上映时间的倒序进行排序

```java
GET movies/_search
{ 
	"query": {  
		"range": {   
			"year": {    
				"gte": 2016,
				"lte": 2018  
			} 
		}
	},
	 "sort": [ {   
		"year": {    
			"order": "desc"  
		} 
	}]
}
```

#### Constant Score

> 查询title中包含有beautiful的所有的电影，不进行相关性算分，查询的数据进行缓存，提高效率

```java
GET movies/_search
{ 
	"query": {  
		"constant_score": {   
			"filter": {    
				"term": {     
					"title": "beautiful"   
				}  
			} 
		}
	}
}
```

### 全文查询

> 全文查询的种类有: Match Query、Match Phrase Query、Query String Query等
>
> 索引和搜索的时候都会进行分词，在查询的时候，会对输入进行分词，然后每个词项会逐个到底层进行
> 查询，将最终的结果进行合并

#### match

> 查询电影名字中包含有beautiful的所有电影，每页十条，取第二页的数据

```java
GET movies/_search
{
 "query": {
  "match": {
   "title": "beautiful"
 }
},
 "from": 10,
 "size": 10
}
```

> 查询电影名字中包含有 beautiful 或者 mind 的所有的数据，但是只查询title和id两个属性

```java
GET movies/_search
{ 
	"_source": ["title", "id"],
	 "query": {  
		"match": {   
			"title": "beautiful mind" 
		}
	}
}
```

#### match_phrase

> 查询电影名字中包含有 "beautiful mind" 这个短语的所有的数据

```java
GET movies/_search
{ 
	"query": {  
		"match_phrase": {   
			"title": "beautiful mind" 
		}
	}
}
```

#### multi_match

> 查询 title 或 genre 中包含有 beautiful 或者 Adventure 的所有的数据

```java
GET movies/_search
{ 
	"query": {  
		"multi_match": {   
			"query": "beautiful Adventure",
			   "fields": ["title", "genre"] 
		}
	}
}
```

#### match_all

> 查询所有的数据

```java
GET movies/_search
{ 
	"query": {  
		"match_all": {}
	}
}
```

#### query_string

> 查询 title 中包含有 beautiful 和 mind 的所有的电影

```java
GET movies/_search
{ 
	"query": {  
		"query_string": {   
			"default_field": "title",
			   "query": "mind AND beautiful" 
		}
	}
}

```

```es
GET movies/_search
{ 
	"query": {  
		"query_string": {   
			"default_field": "title",
			   "query": "mind beautiful",
			   "default_operator": "AND" 
		}
	}
}
```

#### simple_query_string

> simple_query_string 覆盖了很多其他查询的用法。
> 查询 title 中包含有 beautiful 和 mind 的所有的电影

```java
GET movies/_search          

```

```java
GET movies/_search          
{ 
	"query": {  
		"simple_query_string": {   
			"query": "beautiful + mind",
			"fields": ["title"] 
		}
	}
}
```

> 查询title中包含 "beautiful mind" 这个短语的所有的电影 (用法和match_phrase类似)

```java
GET movies/_search
{ 
	"query": {  
		"simple_query_string": {   
			"query": "\"beautiful mind\"",
			"fields": ["title"] 
		}
	}
}
```

> 查询title或genre中包含有 beautiful mind romance 这个三个单词的所有的电影 （与
> multi_match类似）

```java
GET movies/_search
{ 
	"query": {  
		"simple_query_string": {   
			"query": "beautiful mind Romance",
			"fields": ["title", "genre"] 
		}
	}
}
```

> 查询title中包含 “beautiful mind” 或者 "Modern Romance" 这两个短语的所有的电影

```java
GET movies/_search
{ 
	"query": {  
		"simple_query_string": {   
			"query": "\"beautiful mind\" | \"Modern Romance\"",
			   "fields": ["title"] 
		}
	}
}
```

> 查询title或者genre中包含有 beautiful + mind 这个两个词，或者Comedy + Romance +
> Musical + Drama + Children 这个五个词的所有的数据

```java
GET movies/_search
{ 
	"query": {  
		"simple_query_string": {   
			"query": "(beautiful + mind) | (Comedy + Romance + Musical + Drama + Children)",
			"fields": ["title", "genre"] 
		}
	}
}
```

> 查询 title 中包含 beautiful 和 people 但是不包含 Animals 的所有的数据

```java
GET movies/_search
{ 
	"query": {  
		"simple_query_string": {   
			"query": "beautiful + people + -Animals",
			 "fields": ["title"] 
		}
	}
}
```

### 模糊搜索

> 查询title中从第6个字母开始只要最多纠正一次，就与 neverendign 匹配的所有的数据

```java
GET movies/_search
{ 
	"query": {  
		"fuzzy": {   
			"title": {    
				"value": "neverendign",
				    "fuzziness": 1,
				    "prefix_length": 5  
			} 
		}
	}
}
```

### 多条件查询

> 查询title中包含有beautiful或者mind单词，并且上映时间在2016~1018年的所有的电影

```java
GET movies/_search
{ 
	"query": {  
		"bool": {   
			"must": [   {     
				"simple_query_string": {      
					"query": "beautiful mind",
					      "fields": ["title"]    
				}   
			},     {     
				"range": {      
					"year": {       
						"gte": 2016,
						"lte": 2018     
					}    
				}   
			}  ] 
		}
	}
}
```

> 查询 title 中包含有 beautiful 这个单词，并且上映年份在2016~2018年间的所有电影，但是不
> 进行相关性的算分

```java
# filter不会进行相关性的算分，并且会将查出来的结果进行缓存，效率上比 query 高
GET movies/_search
{ 
	"query": {  
		"bool": {   
			"filter": [   {     
				"term": {      
					"title": "beautiful"    
				}   
			},     {     
				"range": {      
					"year": {       
						"gte": 2016,
						       "lte": 2018     
					}    
				}   
			}  ] 
		}
	}
}
```

### 分页查询

```es
{
	"query":{
		"match_all":{}
	},
	"from":0,
	"size":2
}
```



## 二. Mapping

> mapping类似于数据库中的schema，作用如下:
>
> 1. 定义索引中的字段类型；
> 2. 定义字段的数据类型，例如：布尔、字符串、数字、日期.....
> 3. 字段倒排索引的设置

### 数据类型

|       类型名       |                             描述                             |
| :----------------: | :----------------------------------------------------------: |
|    Text/Keyword    | 字符串， Keyword的意思是字符串的内容不会被分词处理，输入是什么内容，存<br/>储在ES中就是什么内容。Text类型ES会自动的添加一个Keyword类型的子字段 |
|        Date        |                           日期类型                           |
| Integer/Float/Long |                           数值类型                           |
|                    |                                                              |

### Mapping的定义

> 语法格式如下：

```es
PUT users
{
  "mappings": {
    // define your mappings here
  }
}
```

### 常见参数

#### index

> 可以给属性添加一个 布尔类型的index属性，标识该属性是否能被倒排索引，也就是说是否能通过该字段进行搜索。

#### null_value

> 在数据索引进ES的时候，当某些数据为 null 的时候，该数据是不能被搜索的，可以使用null_value 属性指定一个值，当属性的值为 null 的时候，转换为一个通过 null_value 指定的值。 null_value属性只能用于Keyword类型的属性

#### analyzer

> 如果想要在创建索引和查询时分别使用不同的分词器，ElasticSearch也是支持的。
>
> 在创建索引，指定analyzer，ES在创建时会先检查是否设置了analyzer字段，如果没定义就用ES预设的
>
> 插入文档时，将text类型的字段做分词然后插入倒排索引，此时就可能用到analyzer指定的分词器

#### search_analyzer

> 在查询时，指定search_analyzer，ES查询时会先检查是否设置了search_analyzer字段，如果没有设置，还会去检查创建索引时是否指定了analyzer，还是没有还设置才会去使用ES预设的
>
> 在查询时，先对要查询的text类型的输入做分词，再去倒排索引搜索，此时就可能用到search_analyzer指定的分词器

#### fields

> 在es中，一个字段可能运用于不同的场景，但是某个字段类型的使用场景是有局限的
>
> fields：多字段特性  让一个字段拥有多个子字段类型，使得一个字段能够被多个不同的索引方式进行索引。

```java
# city能同时被搜索和词语建议补全
PUT index_name
{
  "mappings": {
    "properties": {
      "city": {
        "type": "text",
        "fields": {
          "suggest": {
            "type": "completion",
            "analyzer": "ik_max_word"
          }
        }
      }
    }
  }
}
```

## 三.复杂查询

### 聚合查询

> 聚合允许使用者对 es 文档进行统计分析，类似与关系型数据库中的 group by，当然还有很多其他的聚合，例如取最大值max、平均值avg等等。

> **这里需要注意两点**
>
> (1)、一般情况下,text类型(应为内容较长),es不会为其创建正排索引,但是带有keyword类型的text类型,es会为其创建倒排索引的同时创建正派索引(但是此时的keyword正排索引会有长度限制通过ignore_above去配置)。es中一般只有正排索引才能进行聚合查询
>
> (2)、一般情况下,不会对text字段创建正排索引,应为对大文本字段创建正排索引没有什么意义,而且正排索引会创建磁盘文件,浪费资源和空间.

> 接下来按price字段进行分组：

```es
{
    "size": 0, //关闭hit(source)数据的显示 
	"aggs":{//聚合操作
		"price_group":{//名称，随意起名
			"terms":{//分组
			//分组字段,一般情况下,带有keyword的类型的字段才能进行聚合查询,应为keyword类型,es会为其创建正排索引
				"field":"price.keyword",
				"size": 20,  //显示的桶的个数,常用于分页,
				"order": {
          			"_count": "asc" //按照每个桶统计的数量进行升序排列
        		}
			}
		}
	}
}
```

> 所有手机价格求**平均值**

```java
{
	"aggs":{
		"price_avg":{//名称，随意起名
			"avg":{//求平均
				"field":"price"
			}
		}
	},
    "size":0
}
```



### 推荐搜索

> 在搜索过程中，因为单词的拼写错误，没有得到任何的结果，希望ES能够给我们一个推荐搜索。

```es
GET movies/_search
{ 
	"suggest": {  
	  #title_suggestion为我们自定义的名字  
        "title_suggestion": {   
			"text": "drema",
			   "term": {    
				"field": "title",
				    "suggest_mode": "popular"  
			} 
		}
	}
}
```

> suggest_mode，有三个值：popular、missing、always
> 1. popular 是推荐词频更高的一些搜索。
> 2. missing 是当没有要搜索的结果的时候才推荐。
> 3. always无论什么情况下都进行推荐

### 自动补全

> 自动补全应该是我们在日常的开发过程中最常见的搜索方式了，如百度搜索和京东商品搜索。

![image-20230217151312879](https://gitee.com/studentgitee/note-picture/raw/master/image-20230217151312879.png)



> 自动补全的功能对性能的要求极高，用户每发送输入一个字符就要发送一个请求去查找匹配项。
>
> ES采取了不同的数据结构来实现，并不是通过倒排索引来实现的；需要将对应的数据类型设置为completion ; 所以在将数据索引进ES之前需要先定义 mapping 信息。

#### 定义mapping

```es
{ 
	"movies": {  
		"mappings": {   
			"properties": {    
				"@version": {     
					"type": "text",
					     "fields": {      
						"keyword": {       
							"type": "keyword",
							       "ignore_above": 256     
						}    
					}   
				},
				    "genre": {     
					"type": "text",
					     "fields": {      
						"keyword": {       
							"type": "keyword",
							       "ignore_above": 256     
						}    
					}   
				},
				    "id": {     
					"type": "text",
					     "fields": {      
						"keyword": {       
							"type": "keyword",
							       "ignore_above": 256     
						}    
					}   
				},
				    "title": {     
					"type": "completion"   
				},
				    "year": {     
					"type": "long"   
				}  
			} 
		}
	}
}
```

#### 前缀搜索

```java
GET movies/_search
{ 
	"_source": ["title"],
	 "suggest": {  
		"prefix_suggestion": {   
			"prefix": "Lan",
			   "completion": {    
				"field": "title",
				"skip_duplicates": true,
				"size": 10  
			} 
		}
	}
}
```

![image-20230217151612958](https://gitee.com/studentgitee/note-picture/raw/master/image-20230217151612958.png)

> skip_duplicates: 表示忽略掉重复。
>
> size: 表示返回多少条数据。

### 高亮显示

> 高亮显示在实际的应用中也会碰到很多，如下给出了百度高亮搜索的案例

![image-20230217151717244](https://gitee.com/studentgitee/note-picture/raw/master/image-20230217151717244.png)

> 将所有的包含有 beautiful 的单词高亮显示

```java
GET movies/_search
{ 
	"query": {  
		"match": {   
			"title": "beautiful" 
		}
	},
	 "highlight": {  
		"post_tags": "</span>",
		"pre_tags": "<span color='red'>",
		"fields": {   
		  "title": {} 
		}
	}
}
```

> pre_tags: 是需要高亮文本的前置 html 内容。
>
> post_tags: 是需要高亮的后置html内容。
>
> fields: 是需要高亮的属性。

![image-20230217151829214](https://gitee.com/studentgitee/note-picture/raw/master/image-20230217151829214.png)