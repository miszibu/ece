**ECE自我总结模拟题**

集群1：

集群2：

集群3：

集群4: 

​       





# **Task1

* index books1和books2都是3个主分片，1个副本分片。
* 要求books1只能分配在node1上。
* 要求books2所有分片分配在node2，node3上。
* 集群1 

答案：books1

``` 
GET _cat/nodes?v

GET _cat/shards/book1

PUT books1/_settings
{
  "index.routing.allocation.include._name": "node1",
  "number_of_replicas": 0
}

GET books1/_settings
```

答案：books2

``` 
GET _cat/shards/book2
节点1，配置  node.attr.type: hot
节点2，配置  node.attr.type: warm
节点3, 配置  node.attr.type: warm
重新启动各个节点
PUT books2/_settings
{
  "index.routing.allocation.require.type":"warm"
}
```





# **Task2

* index books3是3个主分片，1个副本分片
* 要求索引强制有意识的分片分配到 不同的rack id上，分别node1的rack是r1，node2和node3的rack是r2.
* 集群1 


答案如下：

定义节点属性

``` 
node1   node.attr.rack: r1
node2   node.attr.rack: r2
node3没有设置
```

设置集群的allocation awareness

``` 
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "rack"
  }
}
```

也可以设置在各个master节点的配置文件中。

```
cluster.routing.allocation.awareness.attributes: rack
```

查看books3的分片情况:

``` 
GET _cat/shards/books3
```





# *Task3

目前有个索引是task3，用oa、OA、Oa、oA phrase查询是3条，使用dingding的phrase查询是2条，通过reindex 索引后能够使得使用oa、OA、Oa、oA、0A、dingding都是6条。

* 准备task3的索引，导入6条不同的文档数据
* reindex task3的索引到task3_new后
* 集群2 

答案如下：

``` 
PUT task3_new
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "whitespace",
          "filter": [
            "custom_filter"
          ]
        }
      },
      "filter": {
        "custom_filter": {
          "type": "synonym",
          "synonyms": [
            "oa,OA,Oa,oA,0A,dingding"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "custom_analyzer"
      }
    }
  }
}
```

进行reindex

```  
POST _reindex
{
  "source":{
    "index":"task3"
  },
  "dest":{
    "index":"task3_new"
  }
}
```

最后进行验证：

``` 
POST task3_new/_search
{
  "query":{
    "match_phrase": {
      "title": "dingding"
    }
  }
}
```

注意使用whitespace的tokenizer.



# *Task4

task4的索引中有两条分别是waynes和wayne's的数据，通过将my_index reindex到task4_new后，使用waynes或wayne's的查询，能返回同样数量的文档和评分。



答案：设置task4_new的setting和mapping

``` 
PUT task4_new
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "cust_char_filter"
          ],
          "tokenizer": "standard"
        }
      },
      "char_filter": {
        "cust_char_filter": {
          "type": "mapping",
          "mappings": [
            "'=>"
          ]
        }
      }
    }
  },
  "mappings":{
    "properties": {
      "title":{
        "type":"text",
        "analyzer": "custom_analyzer"
      }
    }
  }
}
```

执行reindex

``` 
POST _reindex
{
  "source": {
    "index": "task4"
  },
  "dest": {
    "index": "task4_new"
  }
}
```

查询验证：

``` 
POST task4_new/_search
{
  "query":{
    "match":{
      "title": "wayne's"
    }
  }
}
```

第二种简便的方法：

定义下新索引的mapping配置，定义该字段的analyzer为english.

``` 
PUT task4_nn
{
  "mappings": {
    "properties": {
      "title":{
        "type":"text",
        "analyzer": "english"
      }
    }
  }
}
```

随后执行reindex

``` 
POST _reindex
{
  "source":{
    "index":"task4"
  },
  "dest": {
    "index":"task4_nn"
  }
}
```

执行分词器测试

``` 
GET task4_nn/_analyze
{
  "analyzer": "english",
  "text":"i am wayne's"
}
```



# *Task5

索引task5中，有一个数组对象tags，各个数组的元素中，有些有前面有空格，有些后面有空格，通过定义ingest pipeline，将这些数组元素中的空格给去掉。

另外要求，通过reindex将task5这个索引，应用于刚刚定义好的pipeline，从而生成一个新的索引task5_new。



答案如下：

定义一个pipline:

``` 
PUT _ingest/pipeline/pip_1
{
  "description": "this is pip_1",
  "processors": [
    {
      "foreach": {
        "field": "tags",
        "processor": {
          "trim": {
            "field": "_ingest._value"
          }
        }
      }
    }
  ]
}
```

执行reindex 

``` 
POST _reindex
{
  "source":{
    "index":"task5"
  },
  "dest":{
    "index":"task5_new",
    "pipeline": "pip_1"
  }
}
```

做测试

``` 
POST _ingest/pipeline/pip_1/_simulate
{
  "docs": [
    {
      "_source": {
        "tags": [
          "ping pang",
          "basket ball",
          " foot bool "
        ]
      }
    }
  ]
}
```





# *Task6

现有的索引task6中，每个文档都有value01、value02、value03 这三个字段。为这个索引中的所有文档，新增一个字段，名称为newadd，这个字段的值，是value01、value02、value03这三个字段的和(拼接)。



答案如下：

定义一个pipeline

``` 
PUT _ingest/pipeline/pip_2
{
  "description": "this is pip_2",
  "processors": [
    {
      "set":{
        "field":"newadd",
        "value": "{{value01}} {{value02}} {{value03}}"
      }
    }
  ]
}
```

执行update_by_query的API

``` 
POST task6/_update_by_query?pipeline=pip_2
```





# *Task7

为索引task7，设置dynamci mapping，具体的要求如下：

* 一切text类型的字段，类型全部映射为keyword

* 一切以int_开头命名的字段，类型都设置成integer

* bulk导入如下的数据进行验证。

  ``` 
  POST task7/_bulk
  {"index":{"_id":1}}
  {"cont":"你好我好铭毅天下好", "int_value":35}
  {"index":{"_id":2}}
  {"cont":"铭毅天下", "int_value":35}
  {"index":{"_id":3}}
  {"cont":"铭毅好", "int_value":35}
  ```

* 集群2 

答案如下：

设置dynamic mapping

``` 
PUT task7
{
  "mappings": {
    "dynamic_templates": [
      {
        "string_to_keyword": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      },
      {
        "int_to_integer": {
          "match": "int_*",
          "mapping": {
            "type": "integer"
          }
        }
      }
    ]
  }
}
```

随后执行bulk api进行数据的验证。



# *Task8

设置一个index template，符合如下的要求：

* 为log和log-开头的索引，创建3个主分片，1个副本分片。

* 同时为索引创建一个相应的alias

* 使用bulk API，对索引log-001写入多条电影数据

  ``` 
  POST log-001/_bulk
  {"index":{"_id":1}}
  {"name":"movice_001"}
  {"index":{"_id":2}}
  {"name":"movice_002"}
  {"index":{"_id":3}}
  {"name":"movice_003"}
  ```

* 集群2 

答案如下：

设置index tempalte

``` 
PUT _template/temp_1
{
  "index_patterns":["log*","log-*"],
  "settings":{
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "aliases":{
    "alias1":{}
  }
}
```



# Task9

一个单节点的集群，需要开启xpack，设置elastic和kibana的密码都是password，添加一个jian的用户，fullname为Jian, email为aaa@aa.com，授权kibana_user的角色给它。

* 集群3 

答案如下：

``` 
xpack.security.enabled: true
bin/elasticsearch-setup-passwords interactive
修改 kibana的用户配置
```





# *Task10

已有索引task10，里面有个对象类型的字段user，这个user里面还是个数组字段，分为user.first和user.last。先要求定义一个新的索引task10_new中，user字段类型为nested 类型，然后从task10 中reindex数据到task10_new中，使用nested的查询。

* 集群2 

答案如下：

设置新索引中的mapping信息，设置nested的字段

``` 
PUT task10_new
{
  "mappings":{
    "properties": {
      "user":{
        "type":"nested"
      }
    }
  }
}
```

执行reindex操作

``` 
POST _reindex
{
  "source":{
    "index":"task10"
  },
  "dest":{
    "index":"task10_new"
  }
}
```

做nested的查询

``` 
GET task10_new/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "user.first": "li"
              }
            },
            {
              "match": {
                "user.last": "peisi"
              }
            }
          ]
        }
      }
    }
  }
}
```





# *Task11

对movies这个索引，进行bool的should 的查询，要求title字段match  phrase匹配上 haha，并且指定返回的size数量为5，要求结果按照revence字段进行降序排列，并且高亮相应的title字段。高亮的格式是\<strong>  \</strong>。

* 集群2 

答案如下：

``` 
GET movies/_search
{
  "size": 5,
  "query": {
    "bool": {
      "should": {
        "match_phrase": {
          "title": "haha"
        }
      }
    }
  },
  "sort": [
    {
      "revence.keyword": {
        "order": "desc"
      }
    }
  ],
  "highlight": {
    "fields": {
      "title": {
        "pre_tags": [
          "<strong>"
        ],
        "post_tags": [
          "</strong>"
        ]
      }
    }
  }
}
```





# Task12

为集群cluster_one和cluster_two配置跨集群搜索，cluster_one的seed的transport地址是：172.17.0.17:9300，cluster_two的seed的transport地址是：172.17.0.18:9300。

这两个集群中，都有索引kibana_sample_data_flights，同时对这两个集群中的kibana_sample_data_flights索引，进行检索。

* 集群2 
* 集群1 

答案如下：

两个集群设置remote信息

``` 
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster1": {
          "seeds": [
            "10.128.190.86:9300"
          ]
        },
        "cluster2": {
          "seeds": [
            "10.128.190.84:9300"
          ]
        }
      }
    }
  }
}
```

执行跨集群搜索：

``` 
GET /kibana_sample_data_flights,cluster1:kibana_sample_data_flights/_search
{
  "query":{
    "term":{
      "DestWeather":"Rain"
    }
  }
}
```





# *Task13

Suppose you have documents that look like the following:

``` 
{
  "username": "certification@elastic.co",
  "address":{
  	"city": "Mountain View",
  	"state": "California",
  	"country": "United States of America"
  },
  "comment": "To be prepared for the exam, you should be able to complate all of the exam object"
}
```

A Kibana instance for cluster2 is running on port 5602. Define a new index on cluster2 that satisfies the following requirements:

* The name of the index is task13
* The username field is mapped as a keyword only
* The field in address are all mapped as both text and keyword
* The comment field is indexed as text three different ways: using the standard analyzer, the english analyzer, and the dutch analyzer

Index the document above into your new task13 index with an _id of 123.

* 集群2 

答案如下：

定义新索引的mapping信息

``` 
PUT task13
{
  "mappings":{
    "properties": {
      "username":{
        "type":"keyword"
      },
      "address":{
        "properties": {
          "city":{
            "type":"text",
            "fields":{
              "keyword":{
                "type":"keyword"
              }
            }
          },
          "state":{
            "type":"text",
            "fields":{
              "keyword":{
                "type":"keyword"
              }
            }
          },
          "country":{
            "type":"text",
            "fields":{
              "keyword":{
                "type":"keyword"
              }
            }
          }
        }
      },
      "country":{
        "type":"text",
        "fields":{
          "english":{
            "type":"text",
            "analyzer":"english"
          },
          "dutch":{
              "type":"text",
              "analyzer":"dutch"
          }
        }
      }
    }
  }
}
```

随后新增一个文档

``` 
POST task13/_doc/123
{
  "username": "certification@elastic.co",
  "address":{
  	"city": "Mountain View",
  	"state": "California",
  	"country": "United States of America"
  },
  "comment": "To be prepared for the exam, you should be able to complate all of the exam object"
}
```





# *Task14

索引 movie-1，保存的电影信息，title是题目，tags是电影的标签。

* 在title中包含“my”或者“me”。
* 如果在tags中包含"romatic movies"，该条算分提高，如果不包含则算分不变

* 集群2 

答案如下：

``` 
POST movie-1/_search
{
  "query": {
    "bool": {
      "must": {
        "terms": {
          "title": [
            "my",
            "me"
          ]
        }
      },
      "should": [
        {
          "match": {
            "tags": "romatic movies"
          },
          "boosting": 2
        }
      ]
    }
  }
}
```

错误，改为以下的信息：

``` 
POST movie-1/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "terms": {
            "title": ["my","me"]
          }
        }
      ],
      "should": {
        "match": {
          "tags": {
            "query": "romatic movies",
            "boost": 2
          }
        }
      }
    }
  }
}
```





# *Task15

针对食品添加剂food ingredient这个索引为task15，要求添加剂字段

ingredient 这个name包含 tt，符合这个条件的top 10的供应商 manufacturer。

* 集群2 

答案如下：

``` 
POST task15/_search
{
  "query":{
    "match":{
      "ingredient":"tt"
    },
      "aggs":{
        "top_10":{
          "terms":{"field":"manufacturer"},
          "size":10
        }
  }
}
```



# Task16

对索引task16的是三个字段a,b,c进行查询 tt，要求字段c的boost为2，各个字段的查询算法加和。

* 集群2 

答案如下：

``` 
POST task16/_search
{
  "query":{
    "multi_match": {
      "query": "tt",
      "fields": ["a","b^2","c"],
      "type": "most_fields"
    }
  }
}
```





# Task17

对索引task17创建一个快照，在路径/app/ctgcloud/elk/elasticsearch-7.2.1/backups建立快照目录myrepo，创建快照名为snapshot_1。

删除掉索引kibana_sample_data_flights，利用快照进行恢复。

* 集群4 

答案如下：

设置FS目录

``` 
path.repo: ["/app/ctgcloud/elk/elasticsearch-7.2.1/backups"]
```

重启ES

设置snapshot的仓库

``` 
PUT /_snapshot/my_fs_backup
{
    "type": "fs",
    "settings": {
        "location": "/app/ctgcloud/elk/elasticsearch-7.2.1/backups/snapshot_repo",
        "compress": true
    }
}
```

创建snapshot

``` 
PUT /_snapshot/my_fs_backup/snapshot_1?wait_for_completion=true
{
  "indices": "kibana_sample_data_flights",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

删除索引

``` 
DELETE kibana_sample_data_flights
```

利用快照进行恢复

``` 
POST /_snapshot/my_fs_backup/snapshot_1/_restore
```





# *Task18

There is a Kibana instance configured for cluster2 that is running on port 5602. The earthquakes index on cluster2 contains details about earthquakes that occurred for 11 months from January to November of 2016. Write one search that returns both of the following:

* the average magnitude of earthquakes for each of the 11 months
* the month and average magnitude of the month with the largest average magnitude
* the search response does not contain any documents.

In the text field below, provide only the JSON portion of your search request.

* 集群2 

答案如下：

``` 
POST earthquakes/_search
{
  "size":0,
  "aggs":{
    "earthquakes_per_month":{
      "date_histogram": {
        "field": "pt",
        "calendar_interval": "month"
      },
      "aggs":{
        "magiitude_max":{
          "max": {
            "field": "magiitude"
          }
        }
      }
    },
    "max_month_magiitude":{
      "max_bucket": {
        "buckets_path": "earthquakes_per_month>magiitude_max"
      }
    }
  }
}
```

错误，修改为如下：

``` 
POST earthquakes/_search
{
  "size":0,
  "aggs":{
    "earthquakes_per_month":{
      "date_histogram": {
        "field": "pt",
        "calendar_interval": "month"
      },
      "aggs":{
        "magiitude_max":{
          "avg": {
            "field": "magiitude"
          }
        }
      }
    },
    "max_month_magiitude":{
      "max_bucket": {
        "buckets_path": "earthquakes_per_month>magiitude_max"
      }
    }
  }
}
```





# Task19

找出每个月的最大震级magnitude和深度depth。

* 集群2 

答案如下：

``` 
POST earthquakes/_search
{
  "size":0,
  "aggs":{
    "earthquakes_per_month":{
      "date_histogram": {
        "field": "pt",
        "calendar_interval": "month"
      },
      "aggs":{
        "magiitude_max":{
          "max": {
            "field": "magiitude"
          }
        },
        "depth_max":{
          "max":{
            "field": "depth"
          }
        }
      }
    },
    "max_month_magiitude":{
      "max_bucket": {
        "buckets_path": "earthquakes_per_month>magiitude_max"
      }
    },
    "max_month_depth":{
      "max_bucket": {
        "buckets_path": "earthquakes_per_month>depth_max"
      }
    }    
  }
}
```





# *Task20

There is a Kibana instance configured for cluster2 that is running on port 5602. Define a pipeline to be used on the earthquakes index that satisfies the following requirements:

* The ID of the pipeline is earthquakes_pipeline
* The value of each magnitude_type field is uppercased
* If the document does not contain a field named batch_number, then a new batch_number field is added and set to 1
* If the document already contains a field named batch_number, then the batch_number field is incremented by 1

After defining the pipeline, update all of the documents in the earthquakes index, applying your pipeline to each document during the updating.

* 集群2 

答案有些问题：

``` 
PUT _ingest/pipeline/earthquakes_pipeline
{
  "description":"this is earthquakes_pipeline",
  "processors":[
      {
        "uppercase":{
          "field":"magnitude_type"
        }
      },
      {
        "script":{
          "lang":"painless", 
          "source": """
            if (ctx.containsKey("ctx.batch_number") == true)
             { ctx.batch_number += 1;}
            else 
             { ctx.batch_number = 1;}
          """
        }
      }
    ]
  
}
```

答案如下：

``` 
PUT _ingest/pipeline/earthquakes_pipeline
{
  "description":"this is earthquakes_pipeline",
  "processors":[
      {
        "uppercase":{
          "field":"magnitude_type"
        }
      },
      {
        "script":{
          "lang":"painless", 
          "source": """
            if (ctx.containsKey("batch_number") == true)
             { ctx.batch_number += 1;}
            else 
             { ctx.batch_number = 1;}
          """
        }
      }
    ]
}
```





# Task21

There is a Kibana instance configured for cluster2 that is running on port 5602. Write a single search of the movies index on cluster2 that satisfies the following requirements:

* the overview field must contain the phrase "new york"
* at least 2 of the title, tags, or tagline fields must be a match for the phrase "new york"

In the text field below, provide only the JSON portion of your search request.

* 集群2 

答案如下：

``` 
POST movies/_search
{
  "query":{
    "bool":{
      "must":{
        "match_phrase":{
          "overview":"new york"
        }
      },
      "should": [
        {
          "match_phrase": {"title": "new york"}
        },
        {
          "match_phrase": {"tags": "new york"}          
        },
        {
          "match_phrase": {"tagline": "new york"}             
        }
      ],
      "minimum_should_match" : 2
    }
  }
}
```

做一次simulate测试

``` 
POST _ingest/pipeline/earthquakes_pipeline/_simulate
{
  "docs":[
    {
              "_source" : {
          "pt" : "2019-11-01T17:00:00",
          "magiitude" : 8,
          "depth" : 3.212,
          "magnitude_type" : "abc",
          "batch_number": 2
        }
    }
    ]
}
```

执行update_by_query

``` 
POST earthquakes/_update_by_query?pipeline=earthquakes_pipeline
```



# *Task22

在task22中包含一些文档，要求创建索引task22_new，通过reindex api将task22的文档索引到task22_new中。

要求增加一个整型字段field_x，value是task22的title的字符长度；再增加一个数组类型的字段field_y，是value的词集合。

(field_y是空格分割的一组词，比如"foo bar"，索引到task22_new后，要求变成["foo","bar"])

* 集群2 

答案如下：

创建ingest pipeline

``` 
PUT _ingest/pipeline/pip_3
{
  "description":"this is pip_3",
  "processors":[
    {
      "script":{
        "lang":"painless",
        "source": """
          ctx.field_x = ctx.title.length()
        """
      }
    },
    {
      "split":{
        "field":"title",
        "separator":" ",
        "target_field":"field_y"
      }
    }
    ]
}
```

执行reindex

``` 
POST _reindex
{
  "source":{
    "index": "task22"
  },
  "dest": {
    "index":"task22_new",
    "pipeline": "pip_3"
  }
}
```

执行simulate测试

``` 
POST _ingest/pipeline/pip_3/_simulate
{
  "docs":[
    {
      "_source" : {
          "title" : "foo bar"
        }
    }
    ]
}
```





# Task23

为task23设定一个index alias名字为alias2，默认查询只返回评分大于3的电影。

* 集群2 

答案如下：

``` 
POST /_aliases
{
  "actions":[
      {
        "add":{
          "index":"task23",
          "alias": "alias2",
          "filter":{
            "range": {
              "score": {
                "gt": 3
              }
            }
          }
        }
      }
    ]
}
```





# Task24

为task24这个索引，增加一个整形字段length，将索引task24中的一个name字段的字符串长度，计算后写入；将task24索引中的字符串以";"分隔后，写入task24_new这个索引中，horry这个数组字段中。

* 增加一个整形字段，将索引A中的一个字段的字符串长度，计算后写入
* 将A文档中的字符串以";"分隔后，写入索引B中的数组字段中

* 集群2 

答案如下：

定义一个ingest pipeline

```
PUT _ingest/pipeline/pip_4
{
  "description": "this is pip_4",
  "processors": [
    {
      "script": {
        "lang": "painless",
        "source": """
          ctx.length = ctx.name.length()
        """
      }
    },
    {
      "split": {
        "field": "horry",
        "separator": ";"
      }
    }
  ]
}
```

执行simulate测试

``` 
POST _ingest/pipeline/pip_4/_simulate
{
  "docs": [
    {
      "_source": {
        "name": "xiaozhang",
        "horry": "pingpang;basketball;football"
      }
    }
  ]
}
```

最后执行reindex操作

``` 
POST _reindex
{
  "source": {
    "index":"task24"
  },
  "dest":{
    "index":"task24_new",
    "pipeline": "pip_4"
  }
}
```



# Task25

写一个查询，要求某个关键字"new york"在task25这个索引中，4个字段("overview"/"title"/"tags"/"tagline")中至少包含两个以上

* bool查询，should/minimun_should_match

* 集群2 

答案如下：

``` 
POST task25/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "overview": "new york"
          }
        },
        {
          "match": {
            "title": "new york"
          }
        },
        {
          "match": {
            "tags": "new york"
          }
        },
        {
          "match": {
            "tagline": "new york"
          }
        }
      ],
      "minimum_should_match": 2
    }
  }
}
```



# Task26

按照要求写一个search template

* 写入search template，名字为ser_temp_1
* 根据search template写出相应的match query，有"my_field"、"my_value"、"size"字段
* 通过参数来调用这个search template
* 这个search template运用到 kibana_sample_data_flights中
* "my_field"是DestCountry，而"my_value"是CN，"size"为5.
* 集群2 

答案如下:

定义一个search template

``` 
POST _scripts/ser_temp_1
{
  "script":{
    "lang":"mustache",
    "source": {
      "query":{
        "match":{
          "{{my_field}}": "{{my_value}}"
        }
      },
      "size": "{{my_size}}"
    }
  }
}
```

调用这个search template

``` 
GET kibana_sample_data_flights/_search/template
{
  "id":"ser_temp_1",
  "params":{
    "my_field":"DestCountry",
    "my_value":"CN",
    "my_size":"5"
  }
}
```



# Task27

There is a Kibana instance configured for cluster2 that is running on port 5602. Write a single search on the movie_data index on cluster2 that satisfies the following requirements:

* The tags field containes "based on comic book" and "marvel cinematic universe"
* The results are sorted first by budget descending, then release_data descending

In the text field below, provide only the JSON portion of your search request:

* 集群2 

答案如下：

``` 
POST movie_data/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase": {
            "tags": "based on comic book"
          }
        },
        {
          "match_phrase": {
            "tags": "marvel cinematic universe"
          }
        }
      ]
    }
  },
  "sort": [
    {
      "budget": {
        "order": "desc"
      }
    },
    {
      "release_data": {
        "order": "desc"
      }
    }
  ]
}
```



# Task28

对多个索引定义相同的别名，并指定其中的一个索引能写入文档数据。

``` 
开启对别名index，指向hamlet-1
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "hamlet-1",
        "alias": "hamlet",
        "is_write_index": true
      }
    },
    {
      "add": {
        "index": "hamlet-2",
        "alias": "hamlet"
      }
    }
  ]
}
```



# Task29

split一个字段，增加多个字段，每个字段是之前那个字段split后的字段值。

``` 
PUT _ingest/pipeline/split_act_scene_line
{
  "description": "split pipeline",
  "processors": [
    {
      "split": {
        "field": "line_number",
        "separator": "\\."
      }
    },
    {
      "set": {
        "field": "number_act",
        "value": "{{line_number.0}}"
      }
    },
    {
      "set": {
        "field": "number_scene",
        "value": "{{line_number.1}}"
      }
    },
    {
      "set": {
        "field": "number_line",
        "value": "{{line_number.2}}"
      }
    }
  ]
}
```



# 推测考题

* index shard allocation / hot &warm 
* oz同义词
* waynes 和wayne's 
* 多字段的拼接 ，  pipeline set processor + update_by_query
* dynamic template   以x开头的字段，以string类型的字段
* 聚合查询，要么是地震那题，找出最大的深度和震级；要么是食品供应商的那个，找出top 10的供应商
* nested对象的定义和查询
* RBAC
* hightlight + sort
* mulit_match + most_fields
* bool must(terms) + should(增加boost)
* snapshot

