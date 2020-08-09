**题目练习**

摘抄，思考，写下答案，比对答案，再思考，寻求帮助。

第一遍看的时候，不要强制什么都记住，要考虑是否都理解了，是否知识点有遗漏，是否有歧义。



# Deploy and Operate a Cluster

As I began to prepare to the Elastic Certified exam, I felt overwhelmed by the number of resources available on the web for self-study. In addition to the monumental Elastic documentation, I could get lost in all sorts of articles, tutorials, open books, webinars, and course materials. Despite this wealth of information, what I missed most were exercises to practice the six objectives of the certification exam.

I briefly mentioned the shortage of training exercises in a previous article on tips and advice for Elastic Certified Engineer candidate. I also promised to get over the problem in the near future. Well, the future has come! This is the first in a series of blog posts to challenge your preparation for the certification exam and includes various tasks and problems to be solved. Each post will focus on selected Exam Objectives, and today we will start by exercising your ability to operate an Elasticsearch Cluster.

>DISCLAIMER: All exercise in this series are based on the Elasticsearch version currently used for the exam, that is ,v6.5. Please always refer to the Elastic certification FAQs to check the latest exam version.



## Install and Configure Elasticsearch 

There is no better way to test your Elasticsearch skills than having an Elasticsearch instance to work with. Testing the first Exam Objective serves this purpose. By the end of this section, you will have an Elasticsearch cluster running on your local machine, and configured to satisfy a given set of network and security requirements.



### Exerciese 1

In this exercise, you will deploy a cluster of three Elasticsearch nodes with fine-tuned neword and software settings. The code blocks contain information and/or exercise instructions.

```
# ** EXAM OBJECTIVE: INSTALLATION AND CONFIGURATION **
# GOAL: Setup an Elasticsearch cluster that satisfies a given set of requirements
# REQUIRED SETUP: /
```



Let's begin by getting the right Elasticsearch package for your machine and providing an initial setup for each node.

```
# Download the exam version of Elasticsearch
# Deploy the cluster `eoc-01-cluster`, so that it satisfies the following requirements:
# (i) has three nodes, named 'node1', 'node2', and 'node3',
# (ii) all nodes are eligible master nodes
```

答案如下：

```
node.name: node1  
node.master: true
node.data: false
node.ingest: false
node.ml: false
cluster.remote.connect: false
```



Now, let's configure the network and discovery settings.

```
# Bind 'node1' to the IP address "151.101.2.217" and port "9201"
# Bind 'node2' to the IP address "151.101.2.218" and port "9202"
# Bind 'node3' to the IP address "151.101.2.219" and port "9203"
# Configure the cluster discovery module of 'node2' and 'node3' so as to use 'node1' as seed host
```

>The cluster coordination layer of Elasticsearch has been completely rebuilt for v7.x(see the Elastic blog post). As a consequence, the node discovery and cluster formation processes and settings have significant differences than the Elasticsearch exam version(see "Discovery module and its configuration").

答案如下：

```
network.host: 151.101.2.217
http.port: 9201
discovery.seed_hosts: ["node1","node2","node3"]
```



Elaticsearch is a resilient creature, and if a master node goes down, then another eligible master node is promoted. However, in the case of a temporary network patition, this might lead to the promotion of one master node per partition. A cluster with many master nodes? It sounds bad because it is bad. This condition is called a split brain and must be avoied.

Unnecessary in Elasticsearch v7.x(see "Voting Configurations").

```
# Configure the nodes to avoid the split brain scenario.
```

答案如下：

```
cluster.initial_master_node: ["node1","node2","node3"]
```





A node can serve multiple purposes, and you should know how to specify them.

```
# Configure 'node1' to be a data node but not an ingest node
# Configure 'node2' and 'node3' to be both an ingest and data node
```

答案如下：

```
node1的配置：
node.name: node1
node.master: false
node.data: true
node.ingest: false
node.ml: false
cluster.remote.connect: false

node2和node3的配置：
node.name: node2
node.master: false
node.data: true
node.ingest: true
node.ml: false
cluster.remote.connect: false
```



The configuration file of a node supports much more than just Elasticsearch-specific properties. For instance, system and logs configurations, and even index policies.

```
Configure 'node1' to disallow swapping on its host
Configure the JVM settings of each node so that it uses a minimum and maximum of 8GB for the heap
Configure the logging seeings of each node so that 
(i) the logs directory is not the default one,
(ii) the log level for transport-related events is "debug"
Configure the nodes so as to disable the possibility to delete indices using widcard.
```

答案如下：

```
sudo swapoff -a

-Xms8g
-Xmx8g

path.logs: /path/to/logs

PUT /_cluster/settings
{
  "transient": {
    "logger.org.elasticsearch.transport": "debug"
  }
}

action.destructive_requires_name:true

PUT /_cluster/settings
{
    "persistent" : {
       "action.destructive_requires_name":true
    }
}
```



问题:

* discovery.seed_hosts和cluster.initial_master_nodes，这两个参数的配置有什么区别联系呢？

* "http:port: 9203"中间冒号有错误

* Configure the cluster discovery module of `node2` and `node3` so as to use `node1` as seed host.

  discovery.seed_hosts:["node1","node2","node3"], 问题不好理解。

* 设置the log level for transport-related events is "debug"

  ```
  PUT /_cluster/settings
  {
    "transient": {
      "logger.org.elasticsearch.transport": "debug"
    }
  }
  ```

* 参考群主的文档： https://wx.zsxq.com/dweb2/index/group/225224548581?from=mweb&type=detail

* 还有这个提问：https://wx.zsxq.com/dweb2/index/group/225224548581?from=mweb&type=detail

* cluster.remote.connect这个参数有什么作用？

  ```
  cluster.remote.connect: false的设置，是禁止cross-cluster search , 默认是开启的。
  ```

  

* logger.org.elasticsearch.transport这个参数有哪些选项？

  ```
  可以在ES的配置文件中设置logger.org.elasticsearch.transport: debug，也可以在动态调整log级别。采用的是log4j的格式，日志级别从高到低有：ERROR、WARN、INFO、DEBUG。只会打印高于设置级别的日志。
  PUT /_cluster/settings
  {
    "transient": {
      "logger.org.elasticsearch.transport": "debug"
    }
  }
  ```

* action.destructive_requires_name的设置，临时设置和永久设置？

  ```
  PUT /_cluster/settings
  {
      "persistent" : {
         "action.destructive_requires_name":true
      }
  }
  ```

  重启集群后，发现是永久的配置。配置还在的。



### Exerciese 2

In this exercise, you will secure the cluster data using Elasticsearch Security. The exercise requires a running Elasticsearch cluster with at least one node and a Kibana instance. You can spin up such cluster in no time by using a docker-compose file form my elastic-training-repo on Github. So, download the file and run it with docker-compose.

```
# **EXAM OBJECTIVE: INSTALLATION AND CONFIGURATION**
# GOAL: Secure a cluster and an index using Elasticsearch Security 
# REQUIRED SETUP:
# (i)  a running Elasticsearch cluster with at least one one and a Kibana instance,
# (ii) no index with name 'hamlet' is indexed on the cluster
```



Most of the security features of Elasticsearch have been free since v6.8.0 and v7.1.0 (see the Elastic Blog Post). However, the certification exam is currently based on an earlier version, which requires you to first unlock a trial license. This will give you 30 days to enable, setup and evaluate all security settings.

```
# Enable XPack security on the cluster
# Set the password of the 'elastic' and 'kibana' build-in users. Use
# the parttern "{{username}}-password" (e.g., "elastic-password")
# Login to Kibana using the 'elastic' user credentials
```



We are now going to use the bulk API to index some documents into the cluster. The documents are lines from Hamlet by William Shakespeare, and have the following structure:

```
{
  "line_number": "String",
  "speaker": "String",
  "text_entry": "String"
}
```

Let's continue with exercise.

```
# Create the index 'hamlet' and add some documents by running the 
# fllowing _bulk command: 
PUT hamlet/_doc/_bulk
{"index":{"_index":"hamlet","_id":0}}
{"line_number":"1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet","_id":1}}
{"line_number":"2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet","_id":2}}
{"line_number":"3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet","_id":3}}
{"line_number":"4","speaker":"FRANCISCO","text_entry":"Bernardo?"}
{"index":{"_index":"hamlet","_id":4}}
{"line_number":"5","speaker":"BERNARDO","text_entry":"He."}
{"index":{"_index":"hamlet","_id":5}}
{"line_number":"6","speaker":"FRANCISCO","text_entry":"You come most carefully upon your hour."}
{"index":{"_index":"hamlet","_id":6}}
{"line_number":"7","speaker":"BERNARDO","text_entry":"Tis now struck twelve; get thee to bed, Francisco."}
{"index":{"_index":"hamlet","_id":7}}
{"line_number":"8","speaker":"FRANCISCO","text_entry":"For this relief much thanks: tis bitter cold,"}
{"index":{"_index":"hamlet","_id":8}}
{"line_number":"9","speaker":"FRANCISCO","text_entry":"And I am sick at heart."}
{"index":{"_index":"hamlet","_id":9}}
{"line_number":"10","speaker":"BERNARDO","text_entry":"Have you had quiet guard?"}
```



You can specify authentication("who are you") and authorisation("what you can do") policies on the Elaticsearch resources by means of users, roles, and mappings between users and roles. Do you know how to do that?

```
# Create the security role 'francisco_role' in the native realm, so that:
# (i) the role has "monitor" privileges on the cluster,
# (ii) the role has all privileges on the 'hamlet' index
# Create the user 'francisco' with password "francisco-password"
# Assign the role 'francisco_role' to the 'francisco' user 
# Login using the 'francsco' user credentials, and run queries on 
# 'hamlet' to verify that the role privileges were correctly set
```



Not bad, right? Now, let's create a more sophisticated security role, which assigns read-only permissions on indices, documents and fields.

```
# Create the security role 'bernardo_role' in the native realm, 
# so that:
# (i) the role has "monitor" privileges on the cluster,
# (ii) the role has read-only privileges on the 'hamlet' index,
# (iii) the role can see only those documents having "BERNARDO" 
#  as a 'speaker',
# (iv) the role can see only the 'text_entry' field

# Create the user 'bernardo' with password "bernardo-password"
# Assign the role 'bernardo_role' to the 'bernardo' user
# Login using the 'bernardo' user credentials, and run queries on
# 'hamlet' to verify that the role privileges were correctly set.
```

答案如下： 

```
由于Licene不同，是基于请求体来执行的。
POST /_security/role/bernardo_role
{
  "run_as": [ "bernardo" ],
  "cluster": [ "monitor" ],
  "indices": [
    {
      "names": [ "hamlet" ],
      "privileges": [ "read" ],
      "field_security" : {
         "grant" : [ "text_entry" ]
      },
       "query": {"match": {"speaker": "BERNARDO"}}
    }
  ]
}
创建用户
POST /_security/user/bernardo
{
  "password" : "bernardo-password",
  "roles" : [ "bernardo_role"],
  "full_name" : "Jack Nicholson",
  "email" : "test@example.com"
}

POST /_security/role/bernardo_role
{
  "indices": [
    {
      "names": [ "*" ],
      "privileges": [ "read" ],
      "query": {"match": {"speaker": "BERNARDO"}}
    }
  ]
}

POST /_security/role/bernardo_role
{
  "indices": [
    {
      "names": [ "hamlet" ],
      "privileges": [ "read" ],
      "field_security" : {
        "grant" : [ "text_entry" ]
      }
    }
  ]
}
```



Whoops, I asked you to assign the wrong password to the "bernardo" user. My bad, Would you be so kind as to change it?

```
# Change the password of the 'bernardo' user to "poor-bernardo"
```

答案如下：

```
POST /_security/user/bernardo/_password
{
  "password" : "123456"
}
```



## Administer an Elasticsearch Cluster

Elasticsearch ships with reasonable defaults and usually requires little configuration. At the same time, it offers great flexibility to optimise your cluster for high availability, performance, security, performance, and more. An Elastic Cerfiefied Engineer candidate should be very confident with this subject, and the "Cluster Administration" Exam Objectives is there to prove it.



### Exercise 3

In this exercise, you will optimise an Elaticsearch cluster for availability and robustness by configuring how the data is distributed across nodes as indices and shards. Furthermore, you will use the \_cat API to interact with the cluster and check that all your operations have the expected result.

The exercise doesn't require any preliminary set-up. Also, as in Exercise 2, we will use lines from Hamlet by William Shakespeare to index document into the clusters.

```
# **EXAM OBJECTIVE: Cluster adminstration**
# GOAL: Allocate the shards in a way that satisfies a given set of
# requirements
# REQUIRED SETUP: /
```



By now, you should know how to install, deploy and privide a basic configuration to an Elasticsearch node. If not, you can always practice some more with Exercise 1.

```
# Download the exam version of Elasticsearch
# Deploy the cluster 'eoc-06-cluster', with three nodes named 'node1'、'node2', and 'node3'
# Configure the Zen Discovery module of each node so that they can communicate
# Start the cluster
```

> Remember that the node discovery and clsuter formation have significantly changed in Elasticsearch v7.x(see updated "Network Settings")



We're now going to put some date in the cluster, and nothing is better than our good old Hamlet. Let's create two new indices, each with two primary shards and one replica. Aslo, let's add some documnets by using again the \_bulk API.

```
# Create the index 'hamlet-1' with two primary shards and one replica
# Add some documents to 'hamlet-1' by running the command below 
PUT hamlet-1/_doc/_bulk
{"index":{"_index":"hamlet-1","_id":0}}  
{"line_number":"1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet-1","_id":1}} 
{"line_number":"2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet-1","_id":2}}
{"line_number":"3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet-1","_id":3}}
{"line_number":"4","speaker":"FRANCISCO","text_entry":"Bernardo?"}
{"index":{"_index":"hamlet-1","_id":4}}
{"line_number":"5","speaker":"BERNARDO","text_entry":"He."}

# Add some documents to 'hamlet-2' by running the command below
PUT hamlet-2/_doc/_bulk
{"index":{"_index":"hamlet-2","_id":5}}
{"line_number":"6","speaker":"FRANCISCO","text_entry":"You come most carefully upon your hour."}
{"index":{"_index":"hamlet-2","_id":6}}
{"line_number":"7","speaker":"BERNARDO","text_entry":"Tis now struck twelve; get thee to bed, Francisco."}
{"index":{"_index":"hamlet-2","_id":7}}
{"line_number":"8","speaker":"FRANCISCO","text_entry":"For this relief much thanks: tis bitter cold,"}
{"index":{"_index":"hamlet-2","_id":8}}
{"line_number":"9","speaker":"FRANCISCO","text_entry":"And I am sick at heart."}
{"index":{"_index":"hamlet-2","_id":9}}
{"line_number":"10","speaker":"BERNARDO","text_entry":"Have you had quiet guard?"}
```



You can always check the health and other stats of an index by using the cat APIs. For example, verify that all indices'replicas have been alreay allocated -- i.e, their health status is green. And while you are at it, use the right cat API to see the distribution of primary shards and replicas among the nodes(spoiler alert: is the cat shards).

```
# Check that replicas of indices 'hamlet-1' and 'hamlet-2' have been allocated
# Check the distribution of primary shards and replicas of indices 
# 'hamlet-1' and 'hamlet-2' across the nodes of the cluster
```

答案如下：

```
查看所有的索引的分配情况：
GET _cat/shards
GET _cat/shards/ham*?v
查看节点的分配的分片的情况：
GET /_cat/allocation?v
查看有索引的分片未正常分片的原因:
GET /_cluster/allocation/explain
查看每个节点所配置的属性的情况：
GET _cat/nodeattrs
```



Elasticsearch allows you to restrict the set of nodes that can allocate the shards of a given index. This feature is called shard allocation filtering, it can be configured either via APIs or in the configuration file of the node, and it's what we're going to practice with next.

```
# Configure 'hamlet-1' to allocate both primary shards to 'node2', using the node name
# Verify the success of the last action by using the _cat API
# Configure 'hamlet-2' so that no primary shard is allocated to 'node3'
# Verify the success of the last action by using the _cat API
# Remove any allocation filter setting associated with 'hamlet-1' and 'hamlet-2'
```

答案如下：

```
第一种情况，索引hamlet-1分配主分片到node2上。由于只有一个值，这里include和require都是效果都是一样的。
PUT hamlet-1
{
  "settings": {
    "index.routing.allocation.include._name": "node-2"
  }
}
GET _cat/shards
第二种情况，索引hamlet-2不分配主分片到node3上。使用exclude
PUT hamlet-2
{
  "settings": {
    "index.routing.allocation.exclude._name": "node-3"
  }
}
GET _cat/shards
将上面的设置清空：
PUT hamlet-1/_settings
{
    "index.routing.allocation.include._name": null
}
GET hamlet-1/_settings
PUT hamlet-2/_settings
{
    "index.routing.allocation.exclude._name": null
}
GET hamlet-2/_settings
```



Imagine that your cluster is distributed among different locaions(e.g., availability zones,racks,continents,planets!). To increase availability, you want to avoid any data loss in the cluster if no location fails. To this end, you can make Elasticsearch aware of this physical configuration of the cluster so that it can make Elasticsearch aware of this physical configuration of the cluster so that it can distribute shards in such a way as to minimise the impact of failure -- for example, by putting at least one replica of each shard in a different location. How to do that? Does Shard allocation awareness ring any bells ?

```
# Let's assume that we deployed the 'eoc-06-cluster' cluster accross 
# two availability zones, names 'earth' and 'mars'. Add the
# attribute 'AZ' to the nodes configuration, and set its value 
# to "earch" for 'node1' and 'node2', and to "mars" for 'node3'
# Restart the cluster
# Configure the clusters to force shard allocation awareness based on
# the two availability zones, and persist such configuration
# across cluster restarts
# Verify the success of the last action by using the _cat API
```

答案如下：

```
设置
node.attr.AZ: earth
node.name=node1
node.name=node2

###
node.attr.AZ: mars
node.name=node3

设置cluster settings信息
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes":"AZ",
    "cluster.routing.allocation.awareness.force.rack_id.values":"earth,mars"
  }
}
最后可以清除awareness.attributes配置：
PUT /_cluster/settings
{
    "persistent" : {
        "cluster.routing.allocation.awareness.attributes": null
    }
}
```



In the last part of this exercise, we will practice with the last strategy offered by Elasticsearch to control the allocation of shards, also named shard allocation filtering. This strategy allows you to create some of the architectures described in the webinar "Elasticsearch Atchiteture Best Practices", which I strongly encourage you to watch before the exam. Like for the other allocation strategies, you will need to adapt the configuration file of each node and adjust the "index.routing.allocation.*" settings of your indices.

```
# Configure the cluster to reflect a hot/warm architecture, with
# 'node1' as the only hot node
# Configure the 'hamlet-1' index to allocate its shards only to warm nodes
# Verify the success of the last action by using the _cat API
# Remove the hot/warm shard filtering configuration from the 'hamlet-1' configuration
# Let's assume that the nodes have either a "large" or "small" local storage.
# Add the attribute 'storge' to the nodes config, and 
# set its value so that 'node2' is the only with a "small" storage.

# Configure the 'hamlet-2' index to allocate its shards only to 
# nodes with a large storage size
# Verify the success of the last action by using the _cat API
```

答案如下：

```
配置hot/warm节点
##node-1
node.attr.hot_warm_type: hot
##node-2
node.attr.hot_warm_type: warm
PUT hamlet-1
{
  "settings": {
    "index.routing.allocation.include.hot_warm_type": "warm"
  }
}
GET _cat/shards
最后清空配置：
PUT hamlet-1/_settings
{
    "index.routing.allocation.include.hot_warm_type": null
}

配置large、small
##node-1
node.attr.storage: large
##node-2
node.attr.storage: small
PUT hamlet-2
{
  "settings": {
    "index.routing.allocation.include.storage": "large"
  }
}
查看分片未分配的原因：
GET _cluster/allocation/explain
查看分配的分片信息：
GET _cat/shards
查看节点属性值的信息：
GET _cat/nodeattrs?v
```



### Exercise 4

In this exercise, you will spin up a brand new cluster and create a backup repository for (re)stroring snapshots of its data. Also, you will deploy a second cluster and enable cross cluster search with the other one. The exercise doesn't require any preliminary set-up.

```
# ** EXAM OBJECTIVE: Cluster Administration **
# GOAL: Backup and cross-cluster search
# Required Setup: /
```



Let's create a one-node cluster and index some data in it.

```
# Download the exam version of Elasticsearch
# Deploy the cluster 'eoc-06-original-cluster', with one node named 
# 'node-1'
# Start the cluster
# Create the index 'hamlet' and add some documents by running the 
# following _bulk command

PUT hamlet/_doc/_bulk
{"index":{"_index":"hamlet","_id":0}}
{"line_number":"1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet","_id":1}}
{"line_number":"2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet","_id":2}}
{"line_number":"3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet","_id":3}}
{"line_number":"4","speaker":"FRANCISCO","text_entry":"Bernardo?"}
{"index":{"_index":"hamlet","_id":4}}
{"line_number":"5","speaker":"BERNARDO","text_entry":"He."}
```



In Elasticsearch, you can backup your data by creating snapshots of a set of indices or of the entire cluster. Snapshot can be stored either in local repositories or in the storage service of the main cloud providers. Do you know how to take and retore a snapshot? Well, prove it!

```
# Configure 'node-1' to support a shard file system repository for 
# backups located in
# (i) "[home_folder]/repo" and
# (ii) "[home_folder]/elastic/repo" - e.g, "glc/elastic/repo"
# Create the 'hamlet_backup' shared file system repository in 
# "[home_folder]/elastic/repo"
# Create a snapshot of the 'hamlet' index, so that the snapshot
# (i) is named 'hamlet_snapshot_1',
# (ii) is stored into 'hamlet_backup'
# Delete the index 'hamlet'
# Restore the index 'hamlet' using 'hamlet_snapshot_1'
```

答案如下：

```
node-1的集群中设置,用来注册的Repository仓库
path.repo: ["/app/elk/elasticsearch-7.2.1/snapshot/repo", "/app/elk/elasticsearch-7.2.1/snapshot/elastic/repo"]
随后重启节点。
创建快照仓库：
PUT /_snapshot/hamlet_backup
{
	"type": "fs",
	"settings":  {
		"location": "/app/elk/elasticsearch-7.2.1/snapshot/elastic/repo",
		"compress": true
	}
}
创建快照：
PUT /_snapshot/hamlet_backup/hamlet_snapshot_1?wait_for_completion=true
{
	"indices":"hamlet",
	"ignore_unavailable": true,
	"include_global_state": false
}
删除索引：
DELETE hamlet
恢复快照：
POST /_snapshot/hamlet_backup/hamlet_snapshot_1/_restore
{
	"indices": "hamlet",
	"index_settings": {
		"index.number_of_replicas": 5
	},
	"ignore_index_settings": [
		"index.refresh_interval"
	]
}
补充： 
查看所有快照信息：
GET /_snapshot/my_fs_backup/_all
查看正在运行的快照：
GET /_snapshot/my_fs_backup/_current
查看某个特定的快照：
GET /_snapshot/my_fs_backup/snapshot_2
删除快照：
DELETE /_snapshot/my_fs_backup/snapshot_2
注销仓库：
DELETE /_snapshot/my_fs_backup
进行全部索引的恢复：
POST /_snapshot/my_fs_backup/snapshot_2/_restore
```



Now, imagine that you own another Elasticsearch cluster that contains data related to the first one, such as recent adaptations of Hamlet. Also, imagine that you want to run queries against both the original and all the adaptations of the paly. Elasticsearch also offers this functionality, which is called cross-cluster search. To practice with it, we need a second cluster.

```
# Deploy a second cluster 'eoc-06-adaptation-cluster', with one node  named 'node-2'
# Start the cluster
# Create the index 'hamlet-pirate' on 'node-2' and add documnets
# using the _bulk command

PUT hamlet-pirate/_doc/_bulk
{"index":{"_index":"hamlet-pirate","_id":5}}
{"line_number":"6","speaker":"FRANCISCO","text_entry":"Ahoy Matey! Ye come most carefully upon yer hour."}
{"index":{"_index":"hamlet-pirate","_id":6}}
{"line_number":"7","speaker":"BERNARDO","text_entry":"Aye! Tis now struck twelve; get ye to bed, Francisco."}
{"index":{"_index":"hamlet-pirate","_id":7}}
{"line_number":"8","speaker":"FRANCISCO","text_entry":"For this relief much thanks, son of a biscuit eater"}
{"index":{"_index":"hamlet-pirate","_id":8}}
{"line_number":"9","speaker":"BERNARDO","text_entry":"Arrrrrrrrh!"}
```



To enable cross-cluster queries from "eoc-06-adapation-cluster" to "eoc-06-original-cluster", you must configure the latter as a remote cluster of the former one. This can be done either in the configuration file of the node connection to the remote cluster or by using the cluster setting API. Let's do it.

```
# Enable cross cluster search on 'eoc-06-adaptation-cluster', so
# that
# (i) the name of the remote cluster is 'original',
# (ii) the seed is 'node-1', which is listening on the default transport port.
# (iii) the cross cluster configuration persists across multiple restarts
# Run the cross-cluster query below to check your setup

GET /original:hamlet,hamlet-pirate/_search
{
  "query": {
    "match": {
       "speaker": "BERNARDO"
    }
  }
}
```

答案如下：

```
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "original": {
          "seeds": [
            "node-1:9300"
          ]
        },
        "adaptation": {
          "seeds": [
            "node-2:9300"
          ]
        }
      }
    }
  }
}
最后执行查询：
GET /original:hamlet,hamlet-pirate/_search
{
  "query": {
    "match": {
       "speaker": "BERNARDO"
    }
  }
}
```



## Conclusions

This blog post provided four handy exercises to train for two Exam Objectives of the Elastic Certified Engineer exam. In particular, we practised the 'Installation and Configuration' and 'Cluster Administration' objectives. You can find the instructions-only version of the exercises also on this Github repo.

Don't forget that this article was just the beginning of as series! New exercises will be published here on the kreuzwerker blog next week.



# Store Data into ES

This is the second installation of a four-part series of exercises on how to prepare for the Elastic Certified Engineer exam. In the previous blog post, we practised our ability to deploy, configure, and operate an Elasticsearch cluster. Today we'll start working with documents covering the "Indexing Data" Exam Objective of the certification.

Before we begin, let's get a few definitions out of the way. In Elasticsearch, the basic unit of information to persist data is a (JSON) document. Documents with shared purpose and characteristics can be collected into an index. For example, one might have an index for application logs, one for system events, and another fro movie data. The mapping of an index defines the schema of its documents and the way they should be searched. Finally, the process of storing data into Elasticsearch as documents and making it searchable is called indexing.

So much with the theory. Let's get out hands dirty, shall we ?

> DISCLAIMER: All exercise in this series are based on the Elasticsearch version currently used for the exam, that is , v6.5. Please always refer to the Elastic certification FAQs to check the latest exam version.



## Index data

The exercise that fllow will test your ability to create, update and delete indices and documents in Elasticsearch. But before that, we need a running cluster. You can quickly start one off by using the resources in the GitHub repo created for this blog series.



### Exercise 1

Indices APIs and scripts are your best allies to manipulate indices in Elasticsearch. You can practice with both by just following the exercise instructions in the code blocks below. (pro tip: copy-paste the instructions into your Kibana Dev Tools console and work directly from there)

```
# ** EXAM OBJECTIVE: INDEXING DATA**
# GOAL: Create, update and delete indices while satisfying a given 
# set of requirements
# REQUIRED SETUP:
# (i)  a running Elasticsearch cluster with at least one node 
       and a Kibana instance,
# (ii) the cluster has no index with name 'hamlet',
# (iii) the cluster has no template that applies to indices starting by 'hamlet'
```



We start by creating a new index, taking into account its scalability, resiliency, and performance. More precisely, I want you to configure the index with one primary shard in order to avoid oversharding, and two replicas to increase failover and read performance.

Since Elasticsearch v7.x, the number of shards per index is set to one by default(see "Elasticsearch 7.0.0 released").

```
# Create the index 'hamlet-raw' with 1 primary shard and 3 replicas 
```

Now, index your first document in 'hamlet-raw' - of course, within its default type "_doc".

答案如下：

```
创建一个索引，有一个主分片和3个副本分片
PUT hamlet-raw
{
  "settings":{
    "number_of_shards": 1,
    "number_of_replicas": 2
  }
}
```



Indices in Elasticsearch used to support multiple subpartitions called type, which have been deprecated in v7.x and will be completely removed in v8.x. (see "Removal of mapping types"). In Elasticsearch v6.x, an index can have only one type, perferably name _doc.

```
# Add a doucment to 'hamlet-raw', so that the document 
# (i)  has id "1"
# (ii) has default type,
# (iii) has one field name 'line' with value 
#       "To be, or not to be: that is the question"
```

答案如下：

```
指定文档ID，新增一个文档
POST hamlet-raw/_doc/1
{
  "line":"To be, or not to be: that is the question"
}
```



To update the document, you can choose between two APIs, depending on wheather you want to update one document or multiple documents per time.

```
# Update the document with id "1" by adding a field named
# 'line_number' with value "3.1.64"

# Add a new document to 'hamlet-raw', so that the document
# (i)  has the id automatically assigned by Elasticsearch,
# (ii) has default type, 
# (iii) has a field named 'text_entry' with value 
#       "Whether tis nobler in the mind to suffer",
# (iv) has a field name 'line_number' with value '3.1.66'

# Update the last document by setting the value of 'line_number' to "3.1.65"

# In one request, update all documents in 'hanlet-raw' by adding a new
#   field named 'speaker' with value 'Hamlet'
```

第1小问答案如下：

```
对文档ID为1的，新增一个'line_number'字段
POST hamlet-raw/_update/1
{
  "doc":{
    "line_number":"3.1.64"
  }
}
```

第2小问的答案：

```
新增一个文档，自动生成文档ID，有一个字段'text_entry'
POST hamlet-raw/_doc
{
  "text_entry": "Whether tis nobler in the mind to suffer",
  "line_number": "3.1.66"
}
```

第3小问的答案：

```
更新刚才的文档，使得'line_number'字段的值更改为"3.1.65"
POST hamlet-raw/_update/jVYAJ3MB7_FbLxCGn9Ii
{
  "doc":{
    "line_number":"3.1.65"
  }
}
```

第4小问的答案：

```
一次请求，将索引所有文档新增一个字段'speaker'，字段的值为'Hamlet'
方法一：先定义一个ingest pipeline，然后调用update_by_query更新实现
PUT _ingest/pipeline/hamlet-pipeline
{
  "description": "halet-raw index add filed",
  "processors":[
      {
        "set":{
          "field":"speaker",
          "value":"Hamlet"
        }
      }
    ]
}
执行update_by_query进行刷新
POST hamlet-raw/_update_by_query?pipeline=hamlet-pipeline
{
  "query":{
    "bool":{
      "must_not":{
        "exists":{
          "field":"speaker"
        }
      }
    }
  }
}
或者直接执行：POST hamlet-raw/_update_by_query?pipeline=hamlet-pipeline
方法二：直接执行update_by_query，里面增加一个painless script脚本新增字段
POST hamlet-raw/_update_by_query
{
  "script": {
    "source" : "ctx._source.speaker = 'Hamlet'",
    "lang": "painless"
  },
  "query": {
   "match_all":{}
  }
}
或者可以这样：
POST hamlet-raw/_update_by_query
{
  "script": {
    "source" : "ctx._source.speaker = 'Hamlet'",
    "lang": "painless"
  },
  "query":{
    "bool":{
      "must_not":{
        "exists":{
          "field":"speaker"
        }
      }
    }
  }
}
```



The update APIs are more than enough to add new fields to a document or to change some fields' value. However, for more complex manipulation you will need more expressive power, which comes from Painless scripts (pun intended). Painless is a Groovy-style scripting language developed and optimised for Elasticsearch. Here are some script examples to give you some inspiraion to solve the next exercise task.

```
# Update the document with id '1' by renaming the field 'line' into
#   'text_entry'
```

答案如下：

```
先定义个ingest pipeline，用于修改字段名称
PUT _ingest/pipeline/hamlet-rename-field
{
  "description": "doc 1 rename field 'line' into 'text_entry'",
  "processors":[
      {
        "rename":{
          "field":"line",
          "target_field":"text_entry"
        }
      }
    ]
}
然后对hamlet-raw这个索引，去指定update_by_query，对id为1的执行
POST hamlet-raw/_update_by_query?pipeline=hamlet-rename-field
{
  "query": {
    "term": {
      "_id": 1
    }
  }
}
```



We need more data. We are going to use lines from Hamlet by William Shakespeare, structured in documents that specify the line number in the play, the line itself, and the speaker. Let's use the bulk API to create the index.

```
# Create the index 'hamlet' and add some documents by running the 
# following _bulk command 

PUT hamlet/_doc/_bulk
{"index":{"_index":"hamlet","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
{"index":{"_index":"hamlet","_id":4}}
{"line_number":"1.2.2","speaker":"KING CLAUDIUS","text_entry":"The memory be green, and that it us befitted"}
{"index":{"_index":"hamlet","_id":5}}
{"line_number":"1.3.1","speaker":"LAERTES","text_entry":"My necessaries are embarkd: farewell:"}
{"index":{"_index":"hamlet","_id":6}}
{"line_number":"1.3.4","speaker":"LAERTES","text_entry":"But let me hear from you."}
{"index":{"_index":"hamlet","_id":7}}
{"line_number":"1.3.5","speaker":"OPHELIA","text_entry":"Do you doubt that?"}
{"index":{"_index":"hamlet","_id":8}}
{"line_number":"1.4.1","speaker":"HAMLET","text_entry":"The air bites shrewdly; it is very cold."}
{"index":{"_index":"hamlet","_id":9}}
{"line_number":"1.4.2","speaker":"HORATIO","text_entry":"It is a nipping and an eager air."}
{"index":{"_index":"hamlet","_id":10}}
{"line_number":"1.4.3","speaker":"HAMLET","text_entry":"What hour now?"}
{"index":{"_index":"hamlet","_id":11}}
{"line_number":"1.5.2","speaker":"Ghost","text_entry":"Mark me."}
{"index":{"_index":"hamlet","_id":12}}
{"line_number":"1.5.3","speaker":"HAMLET","text_entry":"I will."}
```



Now for the challenging part of the exercise, you're going to create a more complicated script, which modifies documents in a different way depending on the value of a certain field. Also, you are going to store this script in the cluster state, so that you can always reuse it later for updating all documents. If you need some inspiration, have a look at the painless documentation or drop me a comment.

During the exam always test your scripts before applying them to the original index, because, if something goes wrong, you won't have an "undo" option to restore it. As a further precaution, you might also consider to create a backup of the cluster at the beginning of the exam.

```
# Create a script named 'set_is_hamlet' and save it into the cluster state. The script 
#   (i)  adds a field named 'is_hamlet' to each document,
#   (ii) sets the field to "true" if the document has 'speaker' equals to       'HAMLLET'
#   (iii) sets the field to "false" otherwise 

# Update all documents in 'hamlet' by running the 'set_is_hamlet' script
```

答案如下：

```
方法一：
先定义一个painless script
POST /_scripts/set_is_hamlet
{
  "script": {
    "lang": "painless",
    "source": """
     if (ctx._source.speaker == "HAMLET") { 
       ctx._source.is_hamlet = true;
     } else 
     {
       ctx._source.is_hamlet = false;
     } 
"""
  }
}
调用update_by_query进行执行修改
POST hamlet/_update_by_query
{
  "script":{
    "id": "set_is_hamlet"
  }
}
方法二：
直接update_by_query，然后一个DSL做好修改，painless脚本在里面
POST hamlet/_update_by_query
{
   "script": {
    "lang": "painless",
    "source": """
     if (ctx._source.speaker == "HAMLET") { 
       ctx._source.is_hamlet = true;
     } else 
     {
       ctx._source.is_hamlet = false;
     } 
"""
  }
}
方法三：
使用ingest中的pipeline的方法
先定义一个pipeline
PUT _ingest/pipeline/set_is_hamlet
{
    "processors": [
      {
        "script": {
          "source": """
            if (ctx.speaker == 'Hamlet') { ctx.is_hamlet = true; } else  { ctx.is_hamlet = false;}
          """
        }
      }
    ]
}
再执行update_by_query
POST hamlet/_update_by_query?pipeline=set_is_hamlet
{
  "query":{
    "match_all":{}
  }
}
```



Pretty convenient the "update_by_query" API, don't you think? Do you also know how to use its counterpart for deletion?

```
# Remove from 'hamlet' the documents that have either "KING CLAUDIUS" or "LAERTES" as the value of 'speaker'
```

答案如下：

```
POST hamlet/_delete_by_query
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "speaker.keyword": {
              "value": "KING CLAUDIUS"
            }
          }
        },
        {
          "term": {
            "speaker.keyword": {
              "value": "LAERTES"
            }
          }
        }
      ]
    }
  }
}
```



### Exercise 2

Every index in Elasticsearch has its own settings and schema, including such things as the number of shards and replicas, the refresh period, and the type of data you are going to put into it. Of course, you can set this information when you create the index. But if you already know that multiple indices will have some characteristics in common, the best practice is to define an index template that applies to them automatically. A typical use case is time series data such as application logs, with new indices of the same type are created every given period.

```
# ** EXAM OBJECTIVE: INDEXING DATA **
# GOAL: Create index tempaltes that satisfy a given set of requirements
# REQUIRED SETUP:
#  (i)   a running Elasticsearch cluster with at least one node and a Kibana instance,
#  (ii)  the cluster has no index with name 'hamlet',
#  (iii) the cluster has no template that applies to indices starting by 'hamlet'
```



As you may have guessed, this exercise is all about index templates. Let's create and test one, while keeping an eye on the documentation.

```
# Create the index template 'hamlet_template', so that the template
#  (i)  matches any index that starts by "hamlet_" or "hamlet-",
#  (ii)  allocates one primary shard and no replicas for each matching index
# Create the indices 'hamlet2' and 'hamlet_test'
# Verify that only 'hamlet_test' applies the settings defined in 'hamlet_template'
```

答案如下：

```
定义一个index template,以"hamlet_" or "hamlet-"进行匹配，设置主副分片数
PUT /_template/hamlet_template
{
  "index_patterns": ["hamlet_*","hamlet-*"],
  "order": 1,
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```



Index templates are not set in stone but should evolve along with your data. For example, why don't you add a mapping to the template and specify the type of each field? (spoiler alert: we're going to practice with mappings in the next blog post of this series).

```
# Update 'hamlet_template' by defining a mapping for the type "_doc", so that
# (i)  the type has three fields, named 'speaker', 'line_number', and 'text_entry',
# (iii) 'text_entry' uses an "english" analyzer
```

答案如下：

```
设置一个index template，以"hamlet_" or "hamlet-"进行匹配，设置3个字段，并设置其中一个字段的分词器
PUT /_template/hamlet_template
{
  "index_patterns": ["hamlet_*","hamlet-*"],
  "order": 1,
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "speaker": {"type": "text"},
      "line_number":{"type": "long"},
      "text_entry":{
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
```



Updates to an index template are not automatically reflected on the matching indices that already exist. This is because index templates are only applied once at index creation time.

```
# Verify that the updates in 'hamlet_template' did not apply to the existing indices
# In one request, delete both 'hamlet2' and 'hamlet_test'
# Create the index 'hamlet-1' and add some documents by running the following _bulk command 

PUT hamlet-1/_doc/_bulk
{"index":{"_index":"hamlet-1","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet-1","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet-1","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet-1","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}

# Verify that he mapping of 'hamlet-1' is consistent with what defined in 'hamlet_tempalte'
```

答案如下：

```
同时删除索引
DELETE hamlet2/hamlet_test
```



Finally, let's talk about dynamic mapping and dynamic templates. Dynamic mapping is the capability of Elasticsearch to index a document without knowing its schema beforehand. The tool will make an educated guess on the documents'data types, so as to let you work with your data as soon as possible. Although useful during development, you should disable (or restrict) dynamic mapping in production for different reasons. For example it can lead to mapping conflicts, out of memory errors due to  mapping explosion, and -- more in general -- a suboptimal use of your data and resources. Luckily, the alternative to  dynamic mapping is not defining the mapping of every field of every index. Dynamic templates come to the rescue!

```
# Update 'hamlet_template' so as to reject any document having a field that is not deined in the mapping
# Verify that you cannot index the following document in 'hamlet-1' 
PUT hamlet-1/_doc 
{ 
  "author": "Shakespeare" 
} 

# Update 'halet_template' so as enable dynamic mapping again 

# Update 'hamet_template' so as to 
#  (i)  dynamically map to an integer any field that start by 'number_',
#  (ii)  dynamically map to unanalysed text any string field

# Create the index 'hamlet-2' and add a document by runnin the following command
POST hamlet-2/_doc/4
{
  "text_entry": "With turbulent and dangerous lunacy?",
  "line_number": "3.1.4",
  "number_act": "3",
  "speaker": "KING CLAUDIUS"
}
# Verify that the mapping of 'hamlet-2' is consistent with what defined in 'hamlet_tempalte'
```

第1小题答案如下：

```
修改索引模板，对新产生的hamlet-1的索引，新增文档中，存在新添加的字段，ES将报错
PUT /_template/hamlet_template
{
  "index_patterns": [
    "hamlet_*",
    "hamlet-*"
  ],
  "settings": {
    "number_of_shards": "1",
    "number_of_replicas": "0"
  },
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "speaker": {
        "type": "text"
      },
      "line_number": {
        "type": "text"
      },
      "text_entry": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
随后，随意写入规定的字段的文档，是可以正常返回的。也能查询到。
然后，再写入一个新字段的文档，发现会报错了。新索引的mapping，继承了"dynamic": "strict"
POST hamlet-1/_doc/
{
  "id" : 11111111
}
```

第2小题答案：

```
恢复hamlet_template模板可以动态映射新的字段
PUT /_template/hamlet_template
{
  "index_patterns": [
    "hamlet_*",
    "hamlet-*"
  ],
  "settings": {
    "number_of_shards": "1",
    "number_of_replicas": "0"
  },
  "mappings": {
    "dynamic": "true",
    "properties": {
      "speaker": {
        "type": "text"
      },
      "line_number": {
        "type": "text"
      },
      "text_entry": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
```

第3小题答案：

```
修改index template，去匹配以'number_'开头的字段名，字段值为string，map为integer;字符串类型字段都map为不去分词，也就是keyword的类型。
PUT _template/hamlet_template
{
  "index_patterns": [
    "hamlet_*",
    "hamlet-*"
  ],
  "settings": {
      "number_of_shards": "1",
      "number_of_replicas": "0"
  },
  "mappings": {
    "dynamic_templates": [
      {
        "string_as_integer": {
          "match_mapping_type": "string",
          "match": "number_*",
          "mapping": {
            "type": "integer"
          }
        }
      },
      {
        "string_as_keywords": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
GET hamlet-2/_mapping
```



### Exercise 3

This last exercise is not for the faint of heart. You will practice with alases, reindexing, and data pipelines. Roll up your sleeves and have fun!

```
# ** EXAM OBJECTIVE: Indexing Data **
# GOAL: Create an alias, reindex indices, and create data pipelines
# REQUIRED SETUP:
#  (i)  a running Elasticsearch cluster with at least one node and a Kibana instance,
#  (ii) the cluster has no index with name 'hamlet',
#  (iii) the cluster has no template that applies to indices starting by 'hamlet'
```



As usual, let's begin by indexing some data.

```
# Create the indices 'hamlet-1' and 'hamlet-2', each with two primary shards and no replicas
# Add some documents to 'hamlet-1' by running the following command
PUT hamlet-1/_doc/_bulk
{"index":{"_index":"hamlet-1","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet-1","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet-1","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet-1","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}

# Add some documents to 'hamlet-2' by running the following command
PUT hamlet-2/_doc/_bulk
{"index":{"_index":"hamlet-2","_id":4}}
{"line_number":"2.1.1","speaker":"LORD POLONIUS","text_entry":"Give him this money and these notes, Reynaldo."}
{"index":{"_index":"hamlet-2","_id":5}}
{"line_number":"2.1.2","speaker":"REYNALDO","text_entry":"I will, my lord."}
{"index":{"_index":"hamlet-2","_id":6}}
{"line_number":"2.1.3","speaker":"LORD POLONIUS","text_entry":"You shall do marvellous wisely, good Reynaldo,"}
{"index":{"_index":"hamlet-2","_id":7}}
{"line_number":"2.1.4","speaker":"LORD POLONIUS","text_entry":"Before you visit him, to make inquire"}
```

答案如下：

```
PUT hamlet-1
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 2
  }
}

PUT hamlet-2
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 2
  }
}
```



An alias is a secondary name that you can assign to one or more indices. Simple as that, Why should you bother about it? This goes beyond the scope of this article, but let me drop here two click baits: "reindexing with zero downtime", "enhance the usablity of time-based data".

```
# Create the alias 'hamlet' that maps both 'hamlet-1' and 'hamlet-2'
# Verify that the documents grouped by 'hamlet' are 8
```

答案如下：

```
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "hamlet-1",
        "alias": "hamlet"
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
查看文档的数量
GET hamlet/_count
```



By default, if your alias includes more than one index, you cannot index documents using the alias name. But default can be overwritten, if you know how.

```
# Configure 'hamlet-1' to be the write index of the 'hamlet' alias
```

答案如下：

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



Honestly, how much have you enjoyed writing a script in the first exercise? That was rhetorical: you are going to write another one anyhow.

```
# Add a document to 'hamlet', so that document
# (i)  has id "8"
# (ii) has "_doc" type,
# (iii) has a field 'text_entry' with value "With turbulent and dangerous lunacy?"
# (iv)  has a field 'line_number' with value "3.1.4"
# (v)   has a field 'speaker' with value "KING CLAUDIUS"

# Create a script named 'control_reindex_batch' and save it into the cluster state.
# The script checks whether a document has the field "reindexBatch", and
#   (i) in the affirmative case, it increments the field value by a script parameter named 'increment',
#   (ii) otherwise, the script adds the field to the document setting its value to "1"
```

第1小题答案：

```
POST hamlet/_doc/8
{
  "text_entry": "With turbulent and dangerous lunacy?",
  "line_number": "3.1.4",
  "speaker": "KING CLAUDIUS"
}
```

第2小题答案：

```
POST _scripts/control_reindex_batch
{
  "script": {
    "lang": "painless",
  "source": """
      if (ctx._source['reindexBatch'] != null) {
        ctx._source['reindexBatch'] += params.increment;
      } else {
        ctx._source['reindexBatch'] = 1;
      }
    """
  }
}
```



In the first exercise, we could apply the script to all documents of an index by using the update API. That was possible only because we were adding new fields and updating their values, but it won't work if you are trying to remove fields or change their type. Why? Because Elasticsearch wouldn't be certain anymore about how to process the existing data, and your searches would no longer work as expected. How to apply these changes, then? If you are not screaming "REINDEX!" yet, then you should start now.



For a more  efficient reindexing, a best practice is to temporarily configure the destination index to have no replias as well as to disable "index.refresh_interval".

```
# Create the index 'hamlet-new' with 2 primary shards and no replicas.
# Reindex 'hamlet' into 'hamlet-new', while satisfying the fllowing criteria:
#  (i) apply the 'control_reindex_batch' script with the 'increment' parameter set to "1",
#  (ii) reindex using two parallel slices
```

答案如下：

```
PUT hamlet-new
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 0
  }
}
POST _reindex?slices=2
{
  "source": {
    "index": "hamlet"
  },
  "dest": {
    "index": "hamlet-new"
  },
  "script": {
        "id": "control_reindex_batch",
        "params": {
          "increment": 1
        }
      }
}
```



Oh, one more task with index aliases.

```
# In one request, add 'hamlet-new' to the alias 'hamlet' and delete the 'hamlet' and 'hanlet-2' indices
```

答案如下：

```
这个题目有些歧义
POST /_aliases
{
  "actions": [
      {"add": {"index": "hamlet-new", "alias":"hamlet"}},
      {"remove": {"index":"hamlet-1", "alias":"hamlet"}},
      {"remove": {"index":"hamlet-2", "alias":"hamlet"}}
    ]
}
```



The last topic of the "Indexing Data" Exam Objective is to "define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents". An ingest pipeline is a series of processors that enrich and transform data before indexing it into Elasticsearch. For a more narrative introduction to the topic, I recommend this article on the Elastic blog.

```
# Create a pipeline named 'split_act_scene_line'. 
# The pipeline splits the value of 'line_number' using the dots as a separator, and stores the split values into three new fields named 'number_act', 'number_scene', and 'number_line', respectively
```

答案如下：

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



To verify that an ingest pipeline works as expected, you can rely on the simulate pipeline API.

```
# Test the pipeline on the following document
{
	"_source":{
		"line_number": "1.2.3"
	}
}
```

答案如下：

```
POST _ingest/pipeline/_simulate
{
  "pipeline" : {
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
  },
  "docs" : [
    { "_source": { "line_number": "1.2.3"} }
  ]
}
```

参照JAVA的正则表达式：

> https://www.cnblogs.com/hqbhonker/p/7517394.html
>
> \\\S+正则表达式：
>
> https://www.cnblogs.com/jinsdu/p/4526858.html



使用第二种方法，通过split processor先进行对字段值进行切分，然后调用script processor把切分后的各个值，放入到新增的字段中。

```
POST _ingest/pipeline/_simulate
{
  "pipeline" : {
   "description": "split pipeline",
  "processors": [
    {
      "split": {
        "field": "line_number",
        "separator": "\\."
      }
    },
     {
        "script": {
          "source": """
            ctx.number_act = ctx.line_number.0;
            ctx.number_scene = ctx.line_number.1;
            ctx.number_line = ctx.line_number.2;
          """
        }
      }
  ]
  },
  "docs" : [
    { "_source": { "line_number": "1.2.3"} }
  ]
}
```



Satisfied with the outcome? Go update your documents, then !

```
# Update all documents in 'hamlet-new' by using the 'split_act_scene_line' pipeline
```

答案如下：

```
POST hamlet-new/_update_by_query/?pipeline=split_act_scene_line
{}
```



## Conclusions

This blog post offered you three exercise to practice with the "Indexing Data" Exam Objective of the Elastic Certified Engineer exam. As for all the exercises in this series, you can find the instructions -- only version of the exercises also on this Github repo.

Next time we'll focus on mappings and text analyzers. Until then, have a great time!



# Model Data into Elasticsearch

In the previous posts (#1 and #2) of this series, I proposed several exercises in preparation for the Elastic Certified Engineer exam. So far, we have practised operating and indexing data into an Elasticsearch cluster. Today we will work on data mapping and text analysis.

> DISCLAIMER: All exercises in this series are based on the Elasticsearcch version currently used for the exam, that is, v6.5. Please always refer to the Elastic certification FAQs to check the latest exam version.



## Mappings and Text Analysis

The mapping of an index describes the schema of its documents and how to search them. In practice, a mapping tells Elasticsearch what the data types of a document's fields are, which fields are searchable, how to analyse a certain text field, and so forth. This section will help you build confidence with all these settings and features.

Three practical pointers before we start. First, the exercise instructions are those framed into code blocks. I recommend you to copy-paste them into your Kibana Dev Tools and work directly from there. Second, all exercises require a running cluster to work on. Check out my elastic-training-repo on GitHub to start off. Third, our training dataset consists of lines from Hamlet by William Shakespeare, and have the following structure:

```
{
  "line_number": "String",
  "speaker": "String",
  "text_entry": "String"
}
```



### Exercise 1

In this exercise, you will specify a mapping for the training dataset. Nothing too fancy, it's going to be easy.

```
# ** EXAM OBJECTIVE: MAPPINGS AND TEXT ANALYSIS **
# GOAL: Create a mapping that satisfies a given set of requirements 
# REQUIRED SETUP:
#  (i)  a running Elasticsearch cluster with at least one node and a Kibana instance,
#  (ii)  the cluster has no index with name 'hamlet',
#  (iii)  the cluster has no template that applies to indices starting by 'hamlet'
```

答案如下：

```
DELETE hamlet_*
DELETE _template/hamlet*
```



After creating an index, define the type of its fields into the mapping. You can find some examples here.

Indices in Elasticsearch used to support multiple subpartitions called types, which have been deprecated in v7.x and will be completely removed in v8.x (see "Removal of mapping types"). In Elasticsearch v6.x, an index can have only one type, preferably named \_doc.

```
# Create the index 'hamlet-1' with one primary shard and no replicas
# Define a mapping for the default type "_doc" of 'hamlet_1', so that
#  (i)  the type has three fields, named 'speaker', 'line_number', and 'text_entry'.
#  (ii)  'speaker' and 'line_number' are unanalysed strings
```

答案如下：

```
PUT hamlet_1
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "speaker": {
        "type": "keyword"
      },
      "line_number": {
        "type": "keyword"
      },
      "text_entry": {
        "type": "text"
      }
    }
  }
}
```



Further more, I want you to disable aggregations on the "line_number" field, as I couldn't think of any valuable static that we could get out of unique, progressive line number. Note that by disabling aggregations, you are going to save some resources.

```
# Update the mapping of 'hamlet_1' by disabling aggregations on 'line_number'
```

答案如下：

```
DELETE hamlet_1
PUT hamlet_1
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "speaker": {
        "type": "keyword"
      },
      "line_number": {
        "type": "keyword",
        "doc_values": false
      },
      "text_entry": {
        "type": "text"
      }
    }
  }
}
```



Let's add store some data into the index.

```
# Add some documents to 'hamlet_1' by running the following _bulk command
PUT hamlet-1/_doc/_bulk
{"index":{"_index":"hamlet_1","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet_1","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet_1","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet_1","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet     our dear brothers death"}
{"index":{"_index":"hamlet_1","_id":4}}
{"line_number":"1.2.2","speaker":"KING CLAUDIUS","text_entry":"The memory be green,    and that it us befitted"}
```



Do you remember that last time you have changed a datatype in an existing index? You don't, right? That's because it is forbidden for most cases in order to prevent you from applying inconsistent settings to existing data. Some exceptions are adding new fields to the document or a multi-field to a field. For all other cases, the way to update your mapping is by using the Reindex API --- if you came here from my previous blog post, you should have already practised with it. Speaking of multi-fields, this is a pretty neat feature of Elasticsearch, which allows you to index the same field in different ways for different purposes. Elastic published a good webinar about it.

Enough of talking: let's put what we have just discussed into practice.

```
# Create the index 'hamlet_2' with one primary shard and no replicas
# Copy the mapping of 'hamlet_1' into 'hamlet_2', but also define a multi-field for 'speaker'. 
# The name of such multi-field is 'tokens' and its data type is the (default) analysed string 
# Reindex 'hamlet_1' to 'hamlet_2'
# Verify that full-text queries on "speaker.tokens" are enabled on 
'hamlet_2' by running the following command:
GET hamlet_2/_search 
{
  "query": {
    "match": { "speaker.tokens": "hamlet" }
}}
```

答案如下：

```
新创建hamlet_2的mapping和setting
PUT hamlet_2
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "speaker": {
        "type": "keyword",
        "fields": {
          "tokens": {
            "type": "text"
          }
        }
      },
      "line_number": {
        "type": "keyword"
      },
      "text_entry": {
        "type": "text"
      }
    }
  }
}
执行reindex
POST _reindex
{
  "source":{
    "index":"hamlet_1"
  },
  "dest":{
    "index":"hamlet_2"
  }
}
```



### Exercise 2

In the Elasticsearch world, you are highly encouraged to avoid relational data as much as possible, mainly for performance reasons. A nested object in your document slows down your queries significantly. Leave parent/child relationships alone, which, besides the additional indexing complexity, lead to queries up to ten times slower than with nested objects. That said, there might be use cases when such relationships are preferable (see the flowchart below), especially if index-time performance is more important than query-time performance.

The next exercise will make sure that you can model relational data in Elasticsearch.

```
## ** EXAM OBJECTIVE: MAPPING AND TEXT ANALYSIS **
# GOAL: Model relational data
# REQUIRED SETUP:
#  (i)  a running Elasticsearch cluster with at least one node and a Kibana instance,
#  (ii) the cluster has no index with name 'hamlet',
#  (iii) the cluster has no template that applies to indices starting by 'hamlet'
```



We are going to index a new type of document than from the ones we've seen so far. Such documents contain information about Hamlet's characters, notably their name and relationships with other characters in the play. The document structure is as follows:

```
{
  "name": "String",
  "relationship": [
      {
        "name": "String",
        "type": "String"
      }
    ]
}
```



Let's index some data. In particular, let's specify that Hamlet has a friend named Horatio and is the son of Gertrude, and that King Claudius is the uncle of Hamlet.

```
# Create the index 'hamlet_1' with one primary shard and no replicas
# Add some documents to 'hamlet_1' by running the following command
在星球文档中，index的数据中，没有"routing":"abc"
PUT hamlet_1/_doc/_bulk
{"index":{"_index":"hamlet_1","_id":"C0","routing":"abc"}}
{"name":"HAMLET","relationship":[{"name":"HORATIO","type":"friend"},{"name":"GERTRUDE","type":"mother"}]}
{"index":{"_index":"hamlet_1","_id":"C1","routing":"abc"}}
{"name":"KING CLAUDIUS","relationship":[{"name":"HAMLET","type":"nephew"}]}
```

答案如下：

```
DELETE hamlet_3
PUT hamlet_1
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
PUT hamlet_1/_doc/_bulk
{"index":{"_index":"hamlet_1","_id":"C0"}}
{"name":"HAMLET","relationship":[{"name":"HORATIO","type":"friend"},{"name":"GERTRUDE","type":"mother"}]}
{"index":{"_index":"hamlet_1","_id":"C1"}}
{"name":"KING CLAUDIUS","relationship":[{"name":"HAMLET","type":"nephew"}]}
```



How many friends of Gertrude have we defined? None, right? Well, not quite.

```
# Verify that the items of the 'relationship' array cannot be searched independently
# - e.g., searching for a friend named Gertrude will return 1 hit
GET hamlet_1/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "relationship.name": "gertrude" } },
        { "match": { "relationship.type": "friend" } }
      ]
}}}
```



Why is that? Short answer: because of the hack mode by Elasticsearch to store arrays of objects into Lucene. A longer explanation -- and very informative read -- is on the Elastic documentation. To achieve correct cross-object matching, you need to change the datatype of the "relationship" field to "nested".

You cannot change the datatype of a field that has been already mapped. You will need to create a new index with the updated mapping, and reindex the old index to it.

```
# Create the index 'hamlet_2' with one primary shard and no replicas 
# Define a mapping for the default type "_doc" of 'hamlet_2', 
# so that the inner ojects of the 'relationship' field
#  (i)  can be searched independently,
#  (ii) have only unanalzed fields
# Reindex 'hamlet_1' to 'hamlet_2' 

# Verify that the items of the 'relationship' array can now be  searched independently
# e.g., searching for a friend named Gertrude will return no hits
GET hamlet_2/_search 
{
  "query": {
    "nested": {
      "path": "relationship",
      "query": {
        "bool": {
          "must": [
            { "match": { "relationship.name": "gertrude" }},
            { "match": { "relationship.type":  "friend" }} 
          ]
}}}}}
```

第1小题答案：

```
设置mappings和settings
PUT hamlet_2
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings":{
    "properties": {
      "relationship":{
        "type":"nested",
        "properties": {
          "name":{"type":"keyword"},
          "type":{"type":"keyword"}
        }
      }
    }
  }
}
Reindex:
POST _reindex
{
  "source":{
    "index":"hamlet_1"
  },
  "dest":{
    "index":"hamlet_2"
  }
}
```



So far, "hamlet_2" contains only documents related to Hamlet's characters. Let's add more documents representing lines of the play.

```
# Add more documents to 'hamlet_2' by running the following command
星球文档中，同样需要没有"routing"
PUT hamlet_2/_doc/_bulk
{"index":{"_index":"hamlet_2","_id":"L0","routing":"abc"}}
{"line_number":"1.4.1","speaker":"HAMLET","text_entry":"The air bites shrewdly; it is very cold."}
{"index":{"_index":"hamlet_2","_id":"L1","routing":"abc"}}
{"line_number":"1.4.2","speaker":"HORATIO","text_entry":"It is a nipping and an eager air."}
{"index":{"_index":"hamlet_2","_id":"L2","routing":"abc"}}
{"line_number":"1.4.3","speaker":"HAMLET","text_entry":"What hour now?"}
```



A character can have many lines. You can think of it as a one-to-many relationship between character documents and line documents. In Elasticsearch, you can model it as a parent/child relation by using the join datatype, where the character documents play the parent role. I'm going to ask you to model such a relation into the mapping of a new index.

```
# Create the index 'hamlet_3' with only one primary shard and no replicas.
# Copy the mapping of 'hamlet_2' into 'hamlet_3', but also add a join field to define a relation between a 'character' (the parent) and a 'line' (the child). The name of such field is 'character_or_line'
# Reindex 'hamlet_2' to 'hamlet_3'
```

答案如下：

```
创建一个join的父子类型的mapping索引结构 ,如果按照群主的做法，先做一个join的mapping结构，然后再进行index的话，无法把hamlet_2的索引结构完全复制过来，尝试后发现，原有的那个relation字段还是只是text类型。
DELETE hamlet_3
PUT hamlet_3
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "line_number": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "name": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "relationship": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "keyword"
          },
          "type": {
            "type": "keyword"
          }
        }
      },
      "speaker": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "text_entry": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "character_or_line":{
        "type":"join",
        "relations":{"character":"line"}
      }
    }
  }
}
然后再进行reindex
POST _reindex
{
  "source": {
    "index": "hamlet_2"
  },
  "dest": {
    "index": "hamlet_3"
  }
}
```



At the moment, the "character_or_line" relationship exists only in the mapping but not in the data. We are going to create a script for it. The script will update all the line documents associated with a given character, which is specified as a script parameter. The update consists in setting the "character_or_line" join field in such a way as to bind the line documents to the right character document. For example, consider the following line document.

```
{
  "line_number": "1.2.1",
  "speaker": "KING CLAUDIUS",
  "text_entry": "Though yet of Hamlet our dear brothers death"
}
```



Given that the character document of King Claudius has been indexed with id "C1", after the script execution the document will have the new field below: 

```
"charater_or_line": 
{
  "name": "line",
  "parent": "C1"
}
```



All right, let's create and apply the script.

```
# Create a script named 'init_lines' and save it into the cluster state. The script
#  (i) has a parameter named 'charaterID',
#  (ii) adds the field 'character_or_line' to the document,
#  (iii) sets the value of 'character_or_line.name' to 'line',
#  (iv) sets the value of 'character_or_line.parent' to the value of the 'characterID' parameter

# Update the document with id 'CO' (i.e., the character document of Hamlet) by adding the field 'character_or_line' and setting its 'character_or_line.name' value to "character"

# Update the documents in 'hamlet_3' that have "HAMLET" as a 'speaker', by running the 'init_lines' script with 'characterID' set to "CO"
```

第1小问答案如下：

```
创建一个script脚本，这个脚本主要完成对一个索引中，添加子文档的各个字段属性值。
这个索引中，定义的join字段类型已经存在了。
添加的子文档的字段，是两个join类型字段中最关键的两个字段，'character_or_line.name'定义了这个join类型字段中，这篇文档的是子文档，还是父文档。'character_or_line.parent'定义了这个join类型字段中，和这个子文档相关联的父文档的ID是什么。
由于在script中去定义join类型的父子关系文档，需要事先定义一个map对象。
PUT _scripts/init_lines
{
  "script": {
    "lang": "painless",
    "source": """
              HashMap map = new HashMap();
              map.name = 'line';
              map.parent = params.characterId;
              ctx._source.character_or_line = map;
              """
  }
}
```

第2小问答案如下：

```
通过update接口来更新一个父文档的，join字段的相关，父文档的字段属性。
POST hamlet_3/_update/C0
{
  "doc":{
    "character_or_line":{
      "name":"character"
    }
  }
}
```

第3小问答案如下：

```
主要的目的在于，让索引中的某个关联的父文档ID，下面的所有的子文档，完成子文档的字段的赋值，完成子文档的join关系。
题目的条件中，给定了主文档的ID，作为第一步脚本的入参。调用第一步生成的脚本，来完成整个操作。
主要操作的逻辑是，先用query出"speaker"是"hamlet"的文档，然后对这些文档，去调用script脚本，完成这些文档(子文档)，字段值的赋值。
由于query出的各个子文档，默认的routing值都没有设置，而查看ES官方文档说明，在index join类型的子文档的时候，必须去指定routing参数。
这个时候，我们就需要定义一个pipeline，去对查询出来的子文档，修改各个子文档的routing参数，让这些子文档同处于一个routing参数下(即使索引中只有一个分片的情况下)。
"POST update_by_query后面对join子文档去指定routing的参数，好像还是会报错。看官方文档，post后面可以带上routing参数，但是我猜测可能是这里是join类型子文档的缘故吧"
最开始的script脚本中，是无法修改routing的参数，从球友截取的官方文档来看，在script中，routing参数是一个readonly的值。
所以要先定义一个pipeline，去修改文档的routing值。
PUT _ingest/pipeline/set_routing
{
  "processors": [
    {
      "script": {
        "source": """
              ctx._routing = '1';
              """
      }
    }
  ]
}
最后调用update_by_query，有两个目的，一是完成pipeline修改查询出来的文档的routing参数的目的。二是完成调用script脚本，完成子文档中join字段类型中，字段值的赋值操作。
POST hamlet_3/_update_by_query?pipeline=set_routing
{
  "script": {
    "id": "init_lines",
    "params": {
      "characterId": "C0"
    }
  },
  "query": {
    "match": {
      "speaker": "HAMLET"
    }
  }
}
```

验证一下，通过主文档，是否能查询到相应的子文档信息。

```
这里我们通过parent_id的方式来查询，因为这种现在这种情况下，我们父文档的ID号是知道的，就是C0。
POST hamlet_3/_search
{
  "query": {
    "parent_id": {
      "type": "line",
      "id": "C0"
    }
  }
}
也可以通过has_parent的方式来查询子文档，has_parent的方式是，先query查询出主文档，然后根据主文档，来查询出对应的子文档。
POST hamlet_3/_search
{
  "query":{
    "has_parent": {
      "parent_type": "character",
      "query": {
        "match":{
          "name": "HAMLET"
        }
      }
    }
  }
}
```

整个操作的流程，参考文档分享：

> 球友的文档分享: https://blog.csdn.net/qq_42848795/article/details/107194475
>
> 官方文档中关于POST update_by_query中使用了routing参数。
>
> https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update-by-query.html



A join datatype works only if the parent document and all its children are indexed on the same shard (see "Parent-Join Restrictions"). For this reason, when indexing a child document, you typically have to set the routing field with the id of its parent. However, this wasn't necessary for our exercise because we created an index with only one shard.

We can use the has parent query to check whether our script was successful. 

```
# Verify the success of the previous operation using the query below
GET hamlet_3/_search
{
  "query": {
    "has_parent": {
      "parent_type": "character",
      "query": {
        "match": { "name": "HAMLET" }
      }
}}}
```



 ### Exercise 3

Elasticsearch supports several strategies to analyse text fields and  unleash the power of full-text searches. Futher more, it provides a simple, modular mechanism to build analyzers that best fit your needs. In this last exercise, you'll practice with both built-in and custom analyzers.

```
# ** EXAM OBJECTIVE: MAPPING AND TEXT ANALYSIS**
# GOAL: Add built-in text analyzers and specify a custom one
# REQUIRE SETUP:
#  (i)  a running Elasticsearch cluster with at least one node and a Kibana instance,
#  (ii)  the cluster hs no index with name 'hamlet',
#  (iii)  the cluster has no template that applies to indies starting by 'hamlet'
```



Elasticsearch ships with a lot of built-in analyzers to process your text data, Among these, language analyzers take into account the characteristics -- e.g., stopwords, stemming ---- of the specified language. The language supported by Elasticsearch are many, and ---- surprise, surprise ---- English is among them.

```
# Create the index 'hamlet_1' with one primary shard and no replicas
# Define a mapping for the default type "_doc" of 'hamlet_1', so that
#  (i)  the type has three fields, named 'speaker', 'line_number', 'text_entry'
#  (ii) 'text_entry' is associated with the language "english" analyzer
# Add some documents to 'hamlet_1' by running the following command

PUT hamlet_1/_doc/_bulk
{"index":{"_index":"hamlet_1","_id":0}}
{"line_number":"1.1.1","speaker":"BERNARDO","text_entry":"Whos there?"}
{"index":{"_index":"hamlet_1","_id":1}}
{"line_number":"1.1.2","speaker":"FRANCISCO","text_entry":"Nay, answer me: stand, and unfold yourself."}
{"index":{"_index":"hamlet_1","_id":2}}
{"line_number":"1.1.3","speaker":"BERNARDO","text_entry":"Long live the king!"}
{"index":{"_index":"hamlet_1","_id":3}}
{"line_number":"1.2.1","speaker":"KING CLAUDIUS","text_entry":"Though yet of Hamlet our dear brothers death"}
```

答案如下：

```
PUT hamlet_1
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "speaker":{
        "type":"text"
      },
      "line_number":{
        "type":"text"
      },
      "text_entry":{
        "type":"text",
        "analyzer": "english"
      }
    }
  }
}
```



If no build-in analyzer fulfils your needs, you can always create a custom one. As an Elastic Certified Engineer, you are expected to know how to do it.

During the exam, I highly encourage you to test your custom analyzer on synthetic date by using the \_testing API.

```
# Create the index 'hamlet_2' with one primary shard and no replicas.
# Add to 'hamlet_2' a custom analyzer named 'shy_hamlet_analyzer', consisting of 
#  (i)  a char filter to replace the characters "Hamlet" with "[CENSORED]",
#  (ii) a tokenizer to split tokens on whitespaes and columns,
#  (iii) a token filter to ignore any token with less than 5 characters 

# Define a mapping for the default type "_doc" of 'hamlet_2', so that
#  (i)  the type has one field named 'text_entry',
#  (ii)  'text_entry' is associated with the 'shy_hamlet_analyzer' created in the previsous step
```

第1小问的答案：

```
PUT hamlet_2
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "analysis": {
      "analyzer": {
        "shy_hamlet_analyzer": {
          "type": "custom",
          "char_filter": [
            "my_char_filter"
          ],
          "tokenizer": "whitespace",
          "filter": [
            "my_condition"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            "Hamlet => [CENSORED]"
          ]
        }
        },
        "filter": {
          "my_condition": {
            "type": "condition",
            "filter": [
              "lowercase"
            ],
            "script": {
              "source": "token.getTerm().length() < 5"
            }
          }
        }
      }
    },
  "mappings": {
    "properties": {
      "text_entry": {
        "type": "text",
        "analyzer": "shy_hamlet_analyzer"
      }
    }
  }
}
```



You're almost done! The last task is to apply the custom analyzer to real data.

```
# Reindex the 'text_entry' field of 'hamlet_1' into 'hamlet_2'
# Verify that documents have been reindexed to 'hamlet_2' as expected 
# - e.g., by searching for "censored" into the 'text_entry' field
```

答案如下：

```
执行reindex
POST _reindex
{
  "source": {
    "index": "hamlet_1"
  },
  "dest": {
    "index": "hamlet_2"
  }
}
```

使用_analyze的API来进行测试

```
POST hamlet_2/_analyze
{
  "analyzer": "shy_hamlet_analyzer",
  "field":"text_entry", 
  "text": "Though yet of Hamlet our dear brothers death"
}
```

并且可以查询一下，该字段分词后的效果。

```
POST hamlet_2/_search
{
  "query": {
    "match": {
      "text_entry": "[CENSORED]"
    }
  }
}
```



## Conclusions

With the exercises in this blog post you should have a clear overview of the topics covered in the "Mappings and Text Analysis" Exam Objective of the Elastic certification exam. As always, the instructions -  only versions of the exercises are available in this Github repo.

The next blog post of this series wil also be the last one, and we'll focus on queries and aggregations. See you in a few weeks!



# Search and Aggregation

It was a sunny August day when I claimed I would publish the conclusion to this blog series of Elasticsearch exercises "in a few weeks". Time files, doesn't it? Sorry to keep you waiting.

Here is a quick recap: we started by operating and configuring a cluster, then we indexed some data into it, and, finally, we played with mappings and text analysis.

Today, we will practice with searches and aggregations, which are the last two Exam Objectives of the Elastic Certified Engineer exam to be covered.

Disclaimer: All exercises in this series are based on the Elasticsearch version currently used for the exam, that is, v7.2. Please always refer to the Elastic certifiation FAQs to check the latest exam version.



## "You Know, for search"

As the name suggests, searches are the heart of Elasticsearch... Don't get distracted by the sheer amount of features that have been added in the past 10 years; "search" is still its primary function. Follow me in this 2-minutes crash introduction.

Elasticsearch exposes its search capabilities via its rich Query DSL. A query is defined as a JSON object and can contain leaf or compound query clauses. You can use leaf clauses to look into a particular field for some desired values (text,numbers,dates), and use compound clauses to combine multiple queries with logical operators or to alter their behaviour. Each of these clauses can run either in a filter or a query context. You use the filter context when all you want to know is whether a documnet matches a clause. That's a yes or a no. But if you want to know also how well that document matched the clause, then you need the query context to calculate the relevance score (a positive float). The higher the score, the more relevant the document is to the search.

The exercises in this section are designed to assess your proficiency in writing searches for Elasticsearch. But before we start, let's set up your training environment:

1. Spin up your cluster (my GitHub repo might come in handy for this task).
2. Open Kibana and add the three sample data sets that come out of the box.
3. Go to the Kibana Dev Tools page.

And off we go!



### Exercise 1

We start by searching into analysed text fields, such as the message of an application log or the address field of a user profile. The way you do it in Elasticsearch is by writing full-text queries.

You don't remember what is an analysed text field and want to practice with more with it? Check out Exercise 3 of the previous post of this series.

```
# ** EXAM OBJECTIVE: QUERIES **
# GOAL: Create search queries for analyzed text, highlight, pagination, and sort
# REQUIRED SETUP:
#  (i)  a running Elasticsearch cluster with at least one node and a Kibana instance,
#  (ii) add the "Sample web logs" and "Sample eCommerce orders" to Kibana
# Run the next queries on the 'kibana_sample_data_logs' index
```



Let's search the kibana_sample_data_logs index for logs that contain the string "Firefox" in their message. Because the data type of the message field is an analysed text, the standard query for performing full-text searches is the match query.

```
# Search for documents with the 'message' field containing the string "Firefox"
```

答案如下：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "match":{
      "message":"Firefox"
    }
  }
}
```



What do you think would happen if you searched for "firefox" with a lowerase "f"? Nothing, right, because the standard analyzer applied to the message field with lowercase all tokens anyway.



By default, a query response can include up to ten results. But what if you wanted to show up to 50 results? And then what if you wanted to fetch the next 50?

```
# Search for documents with the 'message' field containing the string "Firefox"
    and return (up to ) 50 results.
# As above, but return up to 50 results with  an offset of 50 from the first.
```

第1小问答案如下：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "match":{
      "message":"Firefox"
    }
  },
  "from":0,
  "size":50
}
```



第2小问答案如下：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "match":{
      "message":"Firefox"
    }
  },
  "from":50,
  "size":50
}
```



Deep pagination is highly inefficient when realised using the from and size parameters, as memory consumption and response time grow with the value of the parameters. The best practice is to use the search_after parameter instead.

```
# Search for documents with the 'message' field containing the strings "Firefox" 
   or "Kibana"
```

答案如下：

```
查询的时候加入sort
POST kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {"message": "Firefox"}
        },
        {
          "match": {"message": "Kibana"}
        }
      ]
    }
  },
  "size": 10,
  "sort": [
    {"utc_time": "asc"},
    {"_id": "asc"}
    ]
}
后一个查询中，加入search after
POST kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {"message": "Firefox"}
        },
        {
          "match": {"message": "Kibana"}
        }
      ]
    }
  },
  "size": 10,
  "sort": [
    {"utc_time": "asc"},
    {"_id": "asc"}
    ],
  "search_after": [
    1593317892345,
    "eVZjJHMB7_FbLxCGOHko"
  ]
}
```



Did you write a compound query to fulfil the instruction above? It wasn't necessary. If the match query defines multiple terms (e.g. "firefox kibana"), then any document that matches at least one term is considered a valid response.



And if I want you to search for documents that match all of the query terms? Or two-thirds of them? Good news: you still don't need compound queries, but only match query parameters to configure.

```
# Search for documents with the 'message' field containing both the 
    strings "Firefox" and "Kibana"
```



Did you write a compound query to fulfil the instruction above? It wasn't necessary. If the match query defines multiple terms (e.g., "firefox kibana"), then any document that matches at least one term is considered a valid response.

And if I want to search for documents that match all of the query terms? Or two-thirds of them? Good news: you still don't need compound queries, but only match query parameters to configure.

```
#  Search for documents with the 'message' field containing both the strings "Firefox" and "Kibana"

#  Search for documents with the 'message' field containing at least two of the following strings: "Firefox", "Kibana", "159.64.35.129"
```

第1小问答案：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "match":{
      "message":{
        "query":"Firefox Kibana",
        "operator":"AND"
      }
    }
  }
}
```

第2小题答案：

```
POST kibana_sample_data_logs/_search
{
  "query": {
    "match": {
      "message": {
        "query": "Firefox Kibana 159.64.35.129",
        "minimum_should_match": 2
      }
    }
  }
}
```



When you are searching for a short word in a long text, it won't be easy to spot the exact location of the match in your search results. Opportunely,  Elasticsearch offers highlighters to make matches stand out more clearly. With the highlighting feature enabled, every search hit includes a highlight element where every match is wrapped between - configurable - tags. For example, if I search for "Firefox" and "Kibana", this is what a valid response might look like:

```
"_source" : { 
   "message" : "1.2.3.4 - - [2018-07-22T06:06:42.742Z] GET /kibana.tar.gz HTTP/1.1 200 8489 - Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) Firefox/6.0a1" 
}, 
"highlight" : { 
   "message" : [ 
      "1.2.3.4 - - [2018-07-22T06:06:42.742Z] GET /<em>kibana</em>.tar.gz HTTP/1.1”, 
      "200 8489 - Mozilla/5.0 (X11; Linux x86_64; rv:6.0a1) <em>Firefox</em>/6.0a1" 
  ]}, 
```

OK then, let's highlight something.

```
# Search for documents with the 'message' field containing the string 
     "Firefox" or "Kibana"
# As above, but also return the highlights for the 'message' field
# As above, but also wrap the hightlights in "{{"and"}}"
```

第1小题答案如下：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "match":{
      "message":{
        "query":"Firefox Kibana"
      }
    }
  },
  "highlight": {
    "fields":{
      "message":{"pre_tags": ["<em>"], "post_tags":["</em>"]}
    }
  }
}
```

第2小题答案如下：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "match":{
      "message":{
        "query":"Firefox Kibana"
      }
    }
  },
  "highlight": {
    "fields":{
      "message":{"pre_tags": ["{{"], "post_tags":["}}"]}
    }
  }
}
```



A phrase is an ordered sequence of terms. If you need to search for documents containing that phrase, a match query won't be enough. But there are other full-text queries out there.

```
# Search for documents with the 'message' field containing the
#    phrase "HTTP/1.1 200 51"
```

答案如下：

```
POST kibana_sample_data_logs/_search
{
  "query": {
    "match_phrase": {
      "message": {
        "query": "HTTP/1.1 200 51"
      }
    }
  }
}
```



To conclude, I want you to sort results based on different sort orders and sort models.

```
# Search for documents with the 'message' field containing the 
    phrase "HTTP/1.1 200 51", and sort the results by the 'machine.os' field
    in descending order
    
# As above, but also sort the results by the 'timestamp' field in ascending order.

# Run the next queries on the 'kibana_sample_data_ecommerce' index.
# Search for documents with the 'day_of_week' field containing the string "Monday".
# As above, but sort the results by the 'product.base_price' field.
# in descending order, picking the lowest value of the array
```

第1小问答案：

```
POST kibana_sample_data_logs/_search
{
  "query": {
    "match_phrase": {
      "message": {
        "query": "HTTP/1.1 200 51"
      }
    }
  },
  "sort":[
    {
    "machine.os.keyword":{"order":"desc"}
    }
  ]
}
```

第2小问的答案：

```
POST kibana_sample_data_logs/_search
{
  "query": {
    "match_phrase": {
      "message": {
        "query": "HTTP/1.1 200 51"
      }
    }
  },
  "sort":[
    {
    "machine.os.keyword":{"order":"desc"},
    "timestamp":{"order":"asc"}
    }
  ]
}
```

第3小问的答案：

```
POST kibana_sample_data_ecommerce/_search
{
  "query":{
    "term":{
      "day_of_week": "Monday"
    }
  },
  "sort":[
    {"products.base_price": "desc"}
    ]
}
```



### Exercise 2

We will now practice with term-level queries, searching for precise values in non-analysed fields. Also, we will combine multiple query clauses in more complex compound queries.

```
# ** EXAM OBJECTIVE: QUERIES**
# GOAL: Create search queries for terms, numbers, dates, fuzzy, and compound queries
# REQUIRED SETUP:
#  (i)  a running Elasticsearch cluster with at least one node and a Kibana instance,
#  (ii)  add the "Sample web logs" and "Sample flight data" to Kibana
# Run the next queries on the 'kibana_sample_data_logs' index
```



The log documents in kibana_sample_data_logs don't contain only analysed text fields, but also structured data such as numbers(size in bytes, response codes), dates (timestamp), IP addresses and keywords. Usually, queries on structured data don't contribute to the score of a response, but rather, they say wheather a document should be in the results set or not.

A best practice is to run term-level queries in the filter context. This will speed up search performance because Elasticsearch doesn't need to calculate scores and can cache responses automatically. To run a query clause in the filter context, you need to wrap it into a filter boolen query.

As a start, I want you to search  for log documents with a 4xx Client Error response code. The response code is a number. Range queries, anyone? 

```
# Filter douments with the 'response' field greater or equal to 400 
   and less than 500
# As above, but add a second filter for document with the 'referer'
   field matching "http://twitter.com/success/guion-bluford"
```

答案如下：

```
POST kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "response": {
              "gte": 400,
              "lt": 500
            }
          }
        },
        {
          "term": {
            "referer": "http://twitter.com/success/guion-bluford"
          }
        }
      ]
    }
  }
}
```



Prefix queries allow you to search for that begins with the given sequence of characters. You can use them, for instance, to search for URLs within a particular domain or to find all companies with a name starting with 'kreuzwe'.

If you run a prefix query on an analysed field, then the prefix will be checked against every single token, and not on the string as a whole. As an example, assume that your analysed text field contains "elasticsearch is pure fun". If you run the prefix query "fu" on that field, then you will get a match because of the "fun" token.

To achieve the same behaviour of a prefix query on analysed text, you must use another query.

```
# Filter documents with the 'referer' field that starts by 
    "http://twitter.com/success"
    
# Filter documents with the 'request' field that starts by "/people"
```

第1小问的答案：

```
POST kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "filter": {
        "prefix": {
          "referer": "http://twitter.com/success"
        }
      }
    }
  }
}
```

第2小问的答案：

```
POST kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "filter": {
        "prefix": {
          "request": "people"
        }
      }
    }
  }
}
```



Have you noticed that some documents in the index have the memory field set to null? I want you to write a query to fetch all of them, and then another query that does the exact opposite. Documentation for both cases exists!

```
# Filter documents with the 'memory' field containing any indexed value 

# (opposite of above) Filter documents with the 'memory' field not containing any indexed value
```

第1小问答案：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "bool":{
      "filter":{
        "exists":{
          "field":"memory"
        }
      }
    }
  }
}
也可以使用constant_score filter
POST kibana_sample_data_logs/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "exists": {
          "field": "memory"
        }
      }
    }
  }
}
```

第2小问答案：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "bool":{
      "must_not":{
        "exists":{
          "field":"memory"
        }
      }
    }
  }
}
```



If you have made it to here, you have used boolean queries for writing filters and negations. Let's have a look at the other two boolean types: the logical "or" and "and". Remember that you can nest and combine as many boolean queries as you like, but also remember that this comes with resource and performance costs.

```
# Search for documents with the 'agent' field containing the string
    "Windows" and the 'url' field containing the string "name:john"
    
# As above, but also filter documents with the 'phpmemory' field 
     containing any indexed value
     
# Search for documents that have either the 'response' filed greater
     or equal to 400 or the 'tags' field having the string "error"
     
# Search for documents with the "tags" field that does not contain
     any of the following strings: "warning", "error", "info"
```

第1小问答案：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "bool": {
      "must": [
        {"match":{"agent":"Windows"}},
        {"match_phrase": {
          "url": {"query":"name:john"}
        }}
      ]
    }
  }
```

第2小问答案：

```
POST kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {"agent": "Windows"}
        },
        {
          "match_phrase": {"url":{"query": "name:john"}}
        },
        {
          "exists": {"field": "phpmemory"}
        }
      ]
    }
  }
}
```

第3小问答案：

```
POST kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "range": {
            "response": {
              "gte": 400
            }
          }
        },
        {
          "match":{
            "tags":"error"
          }
        }
      ]
    }
  }
}
```

第4小问答案：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "bool":{
      "must_not": [
        {"match":{"tags":"warning"}},
        {"match":{"tags":"error"}},
        {"match":{"tags":"info"}}
      ]
    }
  }
}
```



Time to dust off range queries again, but now with some date math.

```
# Filter documents with the 'timestamp' field containing a date 
    between today and one week ago
```

答案如下：

```
POST kibana_sample_data_logs/_search
{
  "query":{
    "bool":{
      "filter":[
          {
            "range":{
              "timestamp":{
                "gte":"now-1w",
                "lte":"now"
              }
            }
          }
        ]
    }
  }
}
```



Typos happen, and fuzzy matching is a human-friendly solution to deal with them. Elasticsearch supports fuzziness for full-text and term-level queries. Let's use it on a keyword field.

```
#  Run the next queries on the 'kibana_sample_data_fligts' index 
   Filter  documents with either the 'OriginCityName' or the 'DestCityName'
   fields matching the string "Sydney"
   
#  As above, but allow inexact fuzzy matching, with a maximum allowed 
   "Levenshtein Edit Distance" set to 2.
   Test that the query string "Sydney", "Sidney" and "Sidnei" always return the same 
   number of results.
```

第1小问答案如下：

```
POST kibana_sample_data_flights/_search
{
  "query":{
    "bool":{
      "should":[
          {"term":{"OriginCityName":"Sydney"}},
          {"term":{"DestCityName":"Sydney"}}
        ]
    }
  }
}
```

第2小问答案如下：

```
GET kibana_sample_data_flights/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "fuzzy": {
            "OriginCityName": {
              "value": "Sydney",
              "fuzziness": "2"
            }
          }
        },
        {
          "fuzzy": {
            "DestCityName": {
              "value": "Sydney",
              "fuzziness": "2"
            }
          }
        }
      ]
    }
  }
}
```



### Exercise 3

With this last exercise, we cover what remains of the "Queries" Exam Objective.

```
# ** EXAM OBJECTIVE: QUERIES **
# GOAL: Use scroll API, search templates, script queries
# REQUIRED SETUP:
#   (i) a running Elasticsearch cluster with at least one node and a Kibana instance,
#   (ii)  add the "Sample web logs" and "Sample flight data" to Kibana
```

If you need to fetch a huge number of documents with a single search, increasing the size parameter may not be sufficient (there is a hard limit of 10K hits per response) and for sure is not resource-efficent (deep pagination problem). The recommended solution is to rely on the Elasticsearch scroll API.

```
# Search for all documents in all indices
# As ablove, but use the scroll API to return the first 100 results
#    while keeping the search context alive for 2 minutes
# Use the scroll id included in the response to the previous query
#    and retrieve the next batch of results
```

答案如下：

先使用scroll API查询，会创建一个快照。

```
POST */_search?scroll=2m
{
  "size": 100,
  "query": {
    "match_all": {}
  }
}
```

基于上面查询返回结果中的scroll_id来进行继续查询。

```
POST /_search/scroll
{
  "scroll": "2m",
  "scroll_id": "DnF1ZXJ5VGhlbkZldGNoCQAAAAAABdQBFjlRZjRsN3NBUTBXS0dtZEoxb1JKVVEAAAAAAAXUAhY5UWY0bDdzQVEwV0tHbWRKMW9SSlVRAAAAAAAF1AMWOVFmNGw3c0FRMFdLR21kSjFvUkpVUQAAAAAABdQEFjlRZjRsN3NBUTBXS0dtZEoxb1JKVVEAAAAAAAXUBRY5UWY0bDdzQVEwV0tHbWRKMW9SSlVRAAAAAAAF1AYWOVFmNGw3c0FRMFdLR21kSjFvUkpVUQAAAAAABdQHFjlRZjRsN3NBUTBXS0dtZEoxb1JKVVEAAAAAAAXUCBY5UWY0bDdzQVEwV0tHbWRKMW9SSlVRAAAAAAAF1AkWOVFmNGw3c0FRMFdLR21kSjFvUkpVUQ=="
}
```



To introduce the next topic, let's start with something we already did in Exercise 2.

```
# Run the next queries on the 'kibana_sample_data_logs' index
# Filter documents with the 'response' field greater or equal to 400
```

答案如下：

```
POST kibana_sample_data_logs/_search
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "response": {
            "gte": 400
          }
        }
      }
    }
  }
}
```



If you want to filter only 5xx Server errors, you will need to replace the "400" with a "500", right? Right. And if you want to filter only 4xx Client errors, you will need to add an upper bound to the range query, right? Right. And if you want to add a second filter to select only documnets tagged with "security", you will need to change your query again, right? Right.

Now, imaging that you are developing an application that should support all the cases listed above, possibly more. One way to proceed is to build an ad-hoc query with hard-coded values for each case. Another, much better way is to use search templates and parameterize query values if not entire query clauses.



```
# Create a search tempalte for the above query, so that the template
  (i)   is named "with_reponse_and_tag",
  (ii)  has a parameter "with_min_response" to represent the lower bound of the 'response' field.
  (iii)  has a parameter "with_max_response" to represent the upper bound of the 'response' field.
  (iv)  has a parameter "with_tag" to represent a possible value of the 
'tags' field

# Test the "with_response_and_tag" search template by setting the parameter as follows:
  (i)  "with_min_response":400
  (ii)  "with_max_resposne":500
  (iii)  "with_tag": "security"
```

第1小问答案如下：

``` 
POST _scripts/with_response_and_tag
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "bool": {
          "filter": [
            {
              "range": {
                "response": {
                  "gte": "{{with_min_response}}",
                  "lte": "{{with_max_response}}"
                }
              }
            },
            {
              "match": {
                "tags": "{{with_tag}}"
              }
            }
          ]
        }
      }
    }
  }
}
```

第2小问的答案：

``` 
POST kibana_sample_data_logs/_search/template
{
  "id":"with_response_and_tag",
  "params":{
    "with_min_response":"400",
    "with_max_response":"500",
    "with_tag":"security"
  }
}
```



The search template notation supports some more advanced logic such as concatenation of array values, definition of defaults and conditional clauses. I find the last one particularly useful, and I'll show you why.

``` 
# Update the "with_response_and_tag" search template, so that
   (i)  if the "with_max_response" parameter is not set, then don't set an upper bound to the 'response' value, and
   (ii)  if the "with_tag" parameter is not set, then do not apply that filter at all.

# Test the "with_response_and_tag" search template by setting only the       "with_min_response" parameter to 500.
# Test the "with_response_and_tag" search template by setting the paramters as follows:
  (i)  "with_min_response":500,
  (ii) "with_tag": "security"
```

第1小问答案如下：

```
POST _scripts/with_response_and_tag
{
  "script": {
    "lang": "mustache",
    "source": "{\"query\": {\"bool\": {\"filter\": [{\"range\": {\"response\": {\"gte\": \"{{with_min_response}}\"{{#end}},{{/end}}{{#with_max_response}}\"lte\": \"{{with_max_response}}\"{{/with_max_response}}}}},{{{#with_tag}}\"match\": {\"tags\": \"{{with_tag}}\"}{{/with_tag}}}]}}}"
  }
}
修改后的版本，在{{#with_tag}}前面，加了个空格
POST _scripts/with_response_and_tag
{
  "script": {
    "lang": "mustache",
    "source": "{\"query\": {\"bool\": {\"filter\": [{\"range\": {\"response\": {\"gte\": \"{{with_min_response}}\" {{#with_max_response}},{{/with_max_response}} {{#with_max_response}}\"lte\": \"{{with_max_response}}\"{{/with_max_response}}}}},{ {{#with_tag}}\"match\": {\"tags\": \"{{with_tag}}\"}{{/with_tag}}}]}}}"
  }
}
```

给with_max_response上面加上\{\{\#with_max_response\}\}和\{\{with_max_response\}\};在match 上面加上\{\{\{\#with_tag\}\}和\{\{/with_tag\}\};"source"冒号后面的值从头到尾用""括起来；最后遇到""号前面就加上\，最后转为一行。

第2小问答案：

```
查看渲染的结果：
GET _render/template
{
  "id": "with_response_and_tag",
  "params": {
    "with_min_response": "500"
  }
}
GET _render/template
{
  "id": "with_response_and_tag",
  "params": {
    "with_min_response": "500",
    "with_tag": "security"
  }
}
```



## Aggregations

Not only does Elasticsearch has powerful search capabilities, but it is also extremely efficient in aggregating data to build analytic information. The Elasticsearch documentation groups the aggregations supported into four families:

* Metric, to computer metrics over a set of documents.
* Bucket, to build buckets of documents that meet the same criteria.
* Pipeline, to aggregate the output of other aggregations.
* Matrix, to produce matrix results.



In the only long exercise of this section, you will work with the first three aggregations families, which are the ones required for the Elastic Certified Engineer exam.



### Exercise 4

Straight to the intro!

``` 
# ** EXAM OBJECTIVE: AGGREGATIONS **
# GOAL: Create metrics, bucket, and pipeline aggregations
# REQUIRE SETUP:
#  (i)  a running Elasticsearch cluster with at least one node a Kibana instance,
#  (ii)  add the "Sample flight data" to Kibana
# Run the next queries on the 'kibana_sample_data_flights' index
```



Metrics aggregations can extract values from documents and combine them into metrics. For the sake of this exercise, we will mainly focus on numeric metrics aggregations, but there is also more than that.
```
# Create an aggregation named "max_distance" that calculates the 
  maximun value of the 'DistanceKilometers' field
  
# Create an aggregation named "stats_flight_time" that computes
  stats over the value of the 'FlightTimeMin' field
  
# Create two aggregations, named "cardinality_origin_cities" and 
  "cardinality_dest_cities", that count the distinct values of 
  the 'OriginCityName' and 'DestCityName' fields, respectively
```

第1小问答案：

```
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "max_distance": {
      "max": {
        "field": "DistanceKilometers"
      }
    }
  }
}
```

第2小问答案：

``` 
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "stats_flight_time": {
      "stats": {
        "field": "FlightTimeMin"
      }
    }
  }
}
```

第3小问答案：

```
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "cardinality_origin_cities": {
      "cardinality": {
        "field": "OriginCityName"
      }
    },
    "cardinality_dest_cities": {
      "cardinality": {
        "field": "DestCityName"
      }
    }
  }
}
```



Keep in mind that cardinality aggregations over large dataset always return an approximation.



Bucket aggregations group different documents into buckets based on desired and configurable criteria. Although Elasticsearch V7.2 offers more than 20 bucket aggragations, this exercise covers only a few, starting from the terms and histogram ones.

``` 
# Create an aggregation named "popular_origin_cities" that
  calulates the number of flights grouped by the 'OriginCityName' field
  
# As above, but return only 5 buckets and in descending order

# Create an aggregation named "avg_price_histogram" that groups the 
  documents basesd on 'AvgTicketPrice' by intervals of 250
```

第1小题答案：

```
POST kibana_sample_data_flights/_search
{
  "size":0,
  "aggs":{
    "popular_origin_cities":{
      "terms":{
        "field":"OriginCityName"
      }
    }
  }
}
```

第2小题答案：

```
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "popular_origin_cities": {
      "terms": {
        "field": "OriginCityName",
        "size": 5,
        "order": {
          "_count": "desc"
        }
      }
    }
  }
}
```

第3小题答案：

```
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "avg_price_histogram": {
      "histogram": {
        "field": "AvgTicketPrice",
        "interval": 250
      }
    }
  }
}
```



You can write multiple aggregations within the same request, and you can also nest them. If you are not sure how to do it, this code snippet from the official documentation will help you out.

``` 
#  Create an aggregation named "popular_carriers" that calculates the
     number of flights grouped by the 'Carrier' field
     
#  Add a sub-aggregation to "popular_carriers", named "carrier_stats_delay",
     that computes stats over the value of the 'FlightDelayMin' field
     for the related bucket of carriers
     
#  Add a second sub-aggregation to "popular_carriers", named    "carrier_max_delay", that shows the flight having the maximum value of the 'FlightDelayMin' field for the related bucket of carriers.
```

第1小题答案：

```
POST kibana_sample_data_flights/_search
{
  "size":0,
  "aggs":{
    "popular_carriers":{
      "terms":{
        "field": "Carrier"
      }
    }
  }
}
```

第2小题答案：

``` 
POST kibana_sample_data_flights/_search
{
  "size":0,
  "aggs":{
    "popular_carriers":{
      "terms":{
        "field": "Carrier"
      },
      "aggs":{
        "carrier_stats_delay":{
          "stats":{
            "field": "FlightDelayMin"
          }
        }
      }
    }
  }
}
```

第3小题答案：

``` 
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "popular_carriers": {
      "terms": {
        "field": "Carrier"
      },
      "aggs": {
        "carrier_stats_delay": {
          "stats": {
            "field": "FlightDelayMin"
          }
        },
        "carrier_max_delay": {
          "max": {
            "field": "FlightDelayMin"
          }
        }
      }
    }
  }
}
```



A date histogram aggregation is nothing more than a histogram aggregation specialised for dates. Very handy when you want to observe how certain metrics vary over time.



Since Elasticsearch v7.2, the interval field of the date histogram aggregation has been deprecated in favour of the more explicit calendar_interval and fixed_interval.

``` 
# Use the 'timestamp' field to create an aggregation named
    "flights_every_10_days" that groups the number of flights by an interval of 10 days.

# Use the 'timestamp' field to create an aggregation named "flights_by_day" that groups the number of flights by day.

# Add a sub-aggregation to "flights_by_day", named "destinations_by_day", that groups the day buckets by the value of the "DestCityName" field.
```

第1小问：

``` 
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_every_10_days": {
      "date_histogram": {
        "field": "timestamp",
        "fixed_interval": "10d"
      }
    }
  }
}
```

第2小问：

```
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "day"
      }
    }
  }
}


POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "fixed_interval": "1d"
      }
    }
  }
}
```

第3小问：

``` 
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "day"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          }
        }
      }
    }
  }
}
```



Normally, the response of a bucket aggregation includes only the number of documents into each bucket. If you want to include the most relevant documents in a bucket, you need to add a top hits sub-aggregation.

``` 
# Add a sub-aggregation to the sub-aggregation "destination_by_day",
     named "popular_destinations_by_day", that returns the 3 most popular documents for each bucket (i.e., ordered by their score)

# Update "pupular_destinations_by_day" to display only the 'DestCityName' field in for each top hit object.
```

第1小问答案：

``` 
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "fixed_interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          },
          "aggs": {
            "popular_destinations_by_day": {
              "top_hits": {
                "size": 3
              }
            }
          }
        }
      }
    }
  }
}
```

第2小问答案：

``` 
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "fixed_interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          },
          "aggs": {
            "popular_destinations_by_day": {
              "top_hits": {
                "size": 3,
                "_source": {
                  "includes": [
                    "DestCityName"
                  ]
                }
              }
            }
          }
        }
      }
    }
  }
}
```



Enter the pipeline aggregations: a powerful feature that allows for further computation on the results of other aggregations. The only difficult part of writing a pipeline aggregation is to understand how to specify the input of the pipeline into the buckets_path parameter. I found this summary in the Elastic documentation, along with the follow-up example, particularly useful.



Let's write our pipeline aggregations, and you are free to go!

```
# Remove the "popular_destinations_by_day" sub-sub-aggregation from 
   "flights_by_day"
   
# Add a pipeline aggregation to "flights_by_day", named
   "most_popular_destnation_of_the_day", that identifies the 
   "poular_destinations_by_day" bucket with the most documents for 
   each day.
   
# Add a pipeline aggregation named "day_with_most_flights" that identifies the "flights_by_day" bucket with the most documents.

# Add a pipeline aggregation named 
     "day_with_the_most_popular_destination_over_all_days" that identifies that "flights_by_day" bucket with the largest 
     "most_popular_destination_of_the_day" value.
```

第1小问答案：

"most_popular_destination_of_the_day"，就是统计每一天中，去往哪个城市的航班对多。先做一个按天分桶的"flights_by_day"的聚合，这个聚合中嵌套一个用于统计这一天有多少个目的地的子聚合。最后在外层，配置一个pipeline aggs，用于从这些子聚合中，找寻中，最大数量的子聚合。

```
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "fixed_interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          },
          "aggs": {
            "flight": {
              "cardinality": {
                "field": "FlightNum"
              }
            }
          }
        },
        "most_popular_destination_of_the_day": {
          "max_bucket": {
            "buckets_path": "destinations_by_day>flight"
          }
        }
      }
    }
  }
}
```

第2小问答案：

"day_with_most_flights"，就是统计哪一天有最多个航班数量。先做一个"flights_by_day"按天的分桶，分桶里面嵌套一个"sumflght"聚合，用于统计这一天中有多少个航班(对于这个航班的统计，用去重统计cardinality的方式)。

```
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "fixed_interval": "1d"
      },
      "aggs": {
        "sumflght": {
          "cardinality": {
            "field": "FlightNum"
          }
        }
      }
    },
    "day_with_most_flights": {
      "max_bucket": {
        "buckets_path": "flights_by_day>sumflght"
      }
    }
  }
}
```

第3小问答案：

"day_with_the_most_popular_destination_over_all_days"，意思是基于第一个aggs里面的每一天中的去往哪个城市最多，再多一层聚合，在所有的这些天中，哪一天去往哪个城市最多，前一个聚合侧重的是同一天中哪个城市最多，后一个聚合侧重的是哪一天中哪个城市最多。

``` 
POST kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "flights_by_day": {
      "date_histogram": {
        "field": "timestamp",
        "fixed_interval": "1d"
      },
      "aggs": {
        "destinations_by_day": {
          "terms": {
            "field": "DestCityName"
          },
          "aggs": {
            "flight": {
              "cardinality": {
                "field": "FlightNum"
              }
            }
          }
        },
        "most_popular_destination_of_the_day": {
          "max_bucket": {
            "buckets_path": "destinations_by_day>flight"
          }
        }
      }
    },
    "day_with_the_most_popular_destination_over_all_days": {
      "max_bucket": {
        "buckets_path": "flights_by_day>most_popular_destination_of_the_day"
      }
    }
  }
}
```



## Conclusions

This blog post concludes my series of exercises to prepare for the Elastic Certified Engineer exam. In particular, we covered the two Exam Objectives "Queries" and "Aggregations". The instruction-only version of all exercises is available on this on GitHub repo, which I'll try to keep up-to-date. By the way, you are more than welcome to contribute to the repo, for example, with pull requests of new exercises.

Time to say goodbye. I received very positive feedback from this blog series, and I know for a fact that it helped many people to improve their knowledge and get their certification. I wish you all the same!

Until next time, secure your cluster, and stay Elastic.



# 同义词过滤

## analysis与analyzer的关系

在ES中为了实现analysis，使用的是analyzer来实现的。analysis可以理解为只是定义了一个接口。analyzer可以叫做分析器、分词器。



## analyzer的基本概念

analyzer主要分为三个部分，分别是Character Filter，主要是对分词前的文档词项进行原始文本的处理，例如对于一些网页的数据，去除html; Tokenizers，是重点，就根据不同的规则来进行切分词项；Token Filters，主要是指对切分后的单词进行加工，例如进行转成小写、删除StopWord、增加同义词等内容。



## 同义词创建

同义词的定义，是在token filter中进行定义的。



### 准备些样本数据

在数据中，准备了有oa、oA、OA、0A的字段。

``` 
PUT test_004/_bulk
 {"index":{"_id":1}}
 {"title":"oa is very good"}
 {"index":{"_id":2}}
 {"title":"oA is very good"}
 {"index":{"_id":3}}
 {"title":"OA is very good"}
 {"index":{"_id":4}}
 {"title":"dingding is very good"}
 {"index":{"_id":5}}
 {"title":"dingding is ali software"}
 {"index":{"_id":6}}
 {"title":"0A is very good"}
```



### 定义analyzer(包含同义词)

同义词的设置，其实是在setting中先自定义一个analyzer，然后在mapping 中相应的字段上进行设定刚才定义好的analyzer。

``` 
DELETE task2
PUT /task2
{
  "settings": {
    "index": {
      "analysis": {
        "analyzer": {
          "synonym": {
            "tokenizer": "whitespace",
            "filter": [
              "synonym"
            ]
          }
        },
        "filter": {
          "synonym": {
            "type": "synonym",
            "lenient": true,
            "synonyms": [
              "oa, oA, Oa, OA, 0A, dingding"
            ]
          }
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title":{
        "type": "text", 
        "analyzer": "synonym"
      }
    }
  }
}
```

analysis下面是analyzer，analyzer下面的"synonym"是定义的一个分词器的名字，这个分词器analyzer中，定义了whitespace的tokenizer，filter是token_filter的意思，里面定义了一个自定义的token_filter的名字。

在下面的"filter"包含了"synonym"的定义，这个类型是"synonym"，设置了"synonym"，和相应的所有涉及的同义词， "synonyms": ["oa, oA, Oa, OA, 0A, dingding"]



### 导入数据做验证

先做reindex 

``` 
POST _reindex
{
  "source": {
    "index": "test_004"
  },
  "dest": {
    "index": "task2"
  }
}
```

查看分词的情况：

``` 
POST task2/_analyze
{
  "analyzer": "synonym",
  "text":"dingding is good software"
}
```

再检索下相应的文档，看看是否有定义了同义词的文档，进行了返回。

``` 
GET task2/_search
{
  "query": {
    "match": {
      "title": "dingding"
    }
  }
}
```



# 真题训练一

## 自定义分词插件，让king's和kings有相同的评分

这道题目，和上面的题目的思路是不一样的。上面的思路是token_filter的角度，把各个词项做一个同义的处理。

而这里题目的解决思路是，通过character_filter的角度，直接将king's中的上面的符号去掉，去掉的意思是king's的整个字符都会向前移动了。这样的话，king's的词项就变成了kings.



### 先定义索引的setting和mapping

在setting中定义一个自定义的analyzer，这个analyzer的核心是定义了一个character_filter，目的是去掉'  .

然后在mapping中，对某个字段使用在setting中定义的analyzer.

``` 
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "emoticons"
          ],
          "tokenizer": "standard"
        }
      },
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [
            "' => "
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}
```



### 对索引进行验证

导入必要的测试数据。

``` 
POST my_index/_bulk
{"index":{"_id":1}}
{"title":"kings"}
{"index":{"_id":2}}
{"title":"king's"}
```

对kings和king's进行查询。

``` 
GET my_index/_search
{
  "query": {
    "match": {
      "title": "kings"
    }
  }
}
GET my_index/_search
{
  "query": {
    "match": {
      "title": "king's"
    }
  }
}
```

最后，可以对索引中的某个字段，使用定义的analyzer进行_analyze的接口验证。

``` 
GET my_index/_analyze
{
  "text": "kings",
  "analyzer": "my_custom_analyzer"
}
GET my_index/_analyze
{
  "text": "king's",
  "analyzer": "my_custom_analyzer"
}
```



## 有一个文档，内容类似dog & cat，要求索引这条文档，并且使用match_phrase query，要求dog & cat 或者dog and cat都能match

由于match_phrase的这种query查询的要求的是条件中的词项的顺序，要和设置了分词的那个文档字段中，出现的词项的顺序要一致的情况下，才能匹配。查询条件中的词项要全部出现。

既然如果使用match_phrase 条件是dog & cat, 如果使用标准分词器，那么条件中会分为dog 和cat。如果match_phrase条件是dog and cat，如果使用标准分词器，那么条件中是dog，and，cat。

注意：搜索的时候如果没有特别指定查询分词器，这个分词器就是这个索引这个字段上定义的分词器。不一定就是standard分词器。

方法是，这个索引文档，入索引的时候，就把& 转为and，那么存储在ES中的词项的顺序就是dog and cat。这样match_phrase匹配dog & cat，或者是dog and cat，都能匹配上。

如何将& 转为 and呢，一种是使用character_filter 的方式，将& 映射为 and，这个方法最明显，实际上就是添加了一个and的词项了，使用standard tokenizer或者whitespace tokenizer; 一种是使用token_filter的方式，将&添加同义词为and，这个时候为了能在最后的token_filter中出现，只能使用whitespace tokenzier。



### 方法1：使用character_filter

在索引的setting中自定义一个分词器，然后在对应的mapping字段中，对相应字段设置刚才自定义的分词器。由于使用了character_filter的方式，直接将&转换为and的符号，这样就最省事了。查询的时候，词项所用的分词器，如果没有显性的设置的话，那么就是定义的那个索引中字段的分词器。

``` 
DELETE whitespace_example
PUT /whitespace_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "rebuilt_whitespace": {
          "tokenizer": "whitespace",
           "char_filter": [
            "my_char_filter"
          ]
        }
      },
        "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            "& => and"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title":{
        "type":"text",
        "analyzer": "rebuilt_whitespace"
      }
    }
  }
}
```

导入测试数据

``` 
POST whitespace_example/_bulk
{"index":{"_id":1}}
{"title":"dog & cat"}
```

查询测试

``` 
POST whitespace_example/_search
{
  "query":{
     "match_phrase":{
        "title":"dog & cat"
     }
  }
}
POST whitespace_example/_search
{
  "query":{
     "match_phrase":{
        "title":"dog and cat"
     }
  }
}
```



### 方法2：使用token_filter的方式

同上面是一样，也是在索引的setting中定义一个自定义的分词器，这个自定义的分词器，和上面的不同的是，增加了一个token_filter，将"& => and"，也就是将&增加了一个同义词and，但是这种方式的实现，必须使用whitespace的tokenizer，而不能使用standard tokenizer，因为standard的tokenizer，会将&之前就过滤掉了。

``` 
PUT /test_index009
{
  "settings": {
    "index": {
      "analysis": {
        "analyzer": {
          "synonym": {
            "tokenizer": "whitespace",
            "filter": [
              "synonym"
            ]
          }
        },
        "filter": {
          "synonym": {
            "type": "synonym",
            "lenient": true,
            "synonyms": [
              "& => and"
            ]
          }
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title":{
        "type":"text",
        "analyzer": "synonym"
      }
    }
  }
}
```

添加测试数据：

```
POST test_index009/_bulk
{"index":{"_id":1}}
{"title":"dog & cat"}
```

查询测试：

```
POST test_index009/_search
{
  "query": {
    "match_phrase": {
      "title": "dog & cat"
    }
  }
}
POST test_index009/_search
{
  "query": {
    "match_phrase": {
      "title": "dog and cat"
    }
  }
}
```



## 聚合实现

Task1

There is a Kibana instance configured for cluster2 that is running on port 5602. The earthquakes index on cluster2 contains details about earthquakes that occurred for 11 months from January to November of 2016. Write one search that returns both of the following:

* the average magnitude of earthquakes for each of the 11 months
* the month and average magnitude of the month with the largest average magnitude
* the search response does not contain any documents.

In the text field below, provide only the JSON portion of your search request.

``` 

```

可以看出，题目的要求要满足，求解出每个月的平均地震的级数。外面使用一个pipeline aggs用来求解出这些每个月的平均地震级数中的最大值。同时要求的是，返回的结果中，不返回任何的文档信息。

```
POST earthquakes/_search
{
  "size": 0,
  "aggs":{
    "every_month":{
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs":{
        "avg_mag":{
          "avg":{
            "field": "magnitude"
          }
        }
      }
    },
    "max_month_mags":{
      "max_bucket": {
        "buckets_path": "every_month>avg_mag"
      }
    }
  }
}
```



## 管道实现

Task 3

There is a Kibana instance configured for cluster2 that is running on port 5602. Define a pipeline to be used on the earthquakes index that satisfies the following requirements:

* The ID of the pipeline is earthquakes_pipeline
* The value of each magnitude_type field is uppercased
* If the document does not contain a field named batch_number, then a new batch_number field is added and set to 1
* If the document already contains a field named batch_number, then the batch_number field is incremented by 1

After defining the pipeline, update all of the documents in the earthquakes index, applying your pipeline to each document during the updating.

根据题目的意思，需要创建一个ingest pipeline，指定了这个pipeline的名字。这个pipeline要实现的作用是，将magnitude_type 字段转为大写；这个pipeline中增加一个script processor，这个processor将会去判断现有文档中是否有字段"batch_number"，如果有的话，就把这个字段的值加上1。如果这个字段上没有值的话，就设置默认值为1。

先设置ingest pipeline

``` 
PUT _ingest/pipeline/earthquakes_pipeline
{
  "description": "earthquakes_pipeline",
  "processors": [
    {
      "uppercase": {
        "field": "magnitude"
      }
    },
    {
      "script": {
        "lang": "painless",
        "source": """
              if (ctx.containsKey("batch_number") == true) {
                ctx.batch_number += 1;
              } else {
                ctx.batch_number = 1;
              }
"""
      }
    }
  ]
}
```

做些测试数据

``` 
POST earthquakes/_doc/1
{
  "cont":"1111",
  "magnitude":"asdf"
}
```

最后做一次update_by_query

``` 
POST earthquakes/_update_by_query?pipeline=earthquakes_pipeline
{
  "query":{
    "match_all":{}
  }
}
```



## 查询实现

Task 3

There is a Kibana instance configured for cluster2 that is running on port 5602. Write a single search of the movies index on cluster2 that satisfies the following requirements:

* the overview field must contain the phrase "new york"
* at least 2 of the title, tags, or tagline fields must be a match for the phrase "new york"

In the text field below, provide only the JSON portion of your search request.

题目中要求，搜索的结果中 overview 字段必选包含"new york"，然后title、tags、tagline字段中的至少有两个必须匹配上这个短语"new york".

``` 
POST movies/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase": {
            "overview": "new york"
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "title": "new york"
          }
        },
        {
          "match_phrase": {
            "tags": "new york"
          }
        },
        {
          "match_phrase": {
            "tagline": "new york"
          }
        }
      ],
      "minimum_should_match": 2
    }
  }
}
```



## reindex API

在index_a中包含一些文档，要求创建索引index_b，通过index api将index_a的文档索引到index_b中。

要求增加一个整型字段，value是index_a的field_x的字符长度；再增加一个数组类型的字段，value是field_y的词集合。

(field_y是空格分割的一组词，比如"foo bar"，索引到index_b后，要求变成["foo","bar"])

分析问题：

题目要求使用reindex，但是reindex到新的索引后，要求新索引要增加2个字段。这2个字段的值，一个是某个字段的长度；另外一个字段的值是，将一个字段进行按空格分隔后，形成多个词，以数组的形式存储在新字段中。

这个明显是reindex + ingest pipeline，新增字段可以用set processor或者直接用script  processor，以空格切分text类型的值，以split processor。

答案如下：

``` 
先定义一个index_a
PUT index_a
{
  "mappings": {
    "properties": {
      "title":{
        "type":"keyword"
      }
    }
  }
}
```

插入一些测试数据：

``` 
POST index_a/_bulk
{"index":{"_id":1}}
{"title":"foo bar"}
```

创建ingest pipeline，里面使用script processor和split processor。

``` 
PUT _ingest/pipeline/pipeline-1
{
  "description": "this is a pipleine-1",
  "processors": [
    {
      "script": {
        "lang":"painless",
        "source": """
        ctx.length = ctx.title.length()
        """
      }
    },
    {
      "split":{
        "field":"title",
        "separator": " ",
        "target_field":"field_inb"
      }
    }
  ]
}
```

执行reindex API

``` 
POST _reindex
{
  "source": {
    "index": "index_a"
  },
  "dest": {
    "index": "index_b",
    "pipeline": "pipeline-1"
  }
}
```

最后执行查询验证：

``` 
GET index_b/_mapping
```



# 真题训练二

## 集群部署篇

### 部署3节点的集群，需要同时满足以下要求

* 集群名为"geektime"
* 将每个节点的名字设为和机器名一样，分别为node1、node2、node3
* node1配置成dedicated mater-eligable节点
* node2和node3配置成ingest和data node
* 设置jvm为1g

node1中的参数配置如下：

``` 
jvm.options文件
-Xms1g
-Xmx1g

elasticsearch.yml配置
cluster.name:  geektime
node.name: node-1
network.host: 10.128.190.86
discovery.seed_hosts: ["gzbss190086"]
cluster.initial_master_nodes: ["node-1"]
node.master: true 
node.data: false 
node.ingest: false 
node.ml: false 
cluster.remote.connect: false
```

node2中的参数配置如下：

``` 
jvm.options文件
-Xms1g
-Xmx1g

elasticsearch.yml配置
cluster.name:  geektime
node.name: node-2
network.host: 10.128.190.87
discovery.seed_hosts: ["gzbss190086"]
cluster.initial_master_nodes: ["node-1"]
node.master: false
node.data: true
node.ingest: true
node.ml: false 
cluster.remote.connect: false
```

node3中的参数配置如下：

``` 
jvm.options文件
-Xms1g
-Xmx1g

elasticsearch.yml配置
cluster.name:  geektime
node.name: node-3
network.host: 10.128.190.88
discovery.seed_hosts: ["gzbss190086"]
cluster.initial_master_nodes: ["node-1"]
node.master: false
node.data: true
node.ingest: true
node.ml: false 
```



### 配置3节点的集群，加上一个kibana的实例，设定以下安全防护

* 为集群配置basic authentication
* 将kibana连接到Elasticsearch
* 创建一个名为geektime的用户
* 创建一个名为orders的索引
* geektime用户只能读取和写入orders的索引，不能删除及修改orders

由于考试的时候，使用的是白金版的ES，不需要配置证书，只需要集群的每个节点之间配置如下参数，开启RBAC。

``` 
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```

随后启动ES，设置各个内置的账号的用户和密码。

``` 
bin/elasticsearch-setup-passwords interactive 
```

事先创建模拟的索引和数据。

``` 
PUT orders/_bulk
{"index":{"_id":1}}
{"name":"11111"}
{"index":{"_id":2}}
{"name":"22222"}
```

通过API在ES中创建相应的角色。

``` 
POST /_security/role/geektime_role
{
  "indices": [
    {
      "names": [ "orders" ],
      "privileges": ["read","write"]
    }
  ]
}
```

创建geektime_role这个用户，分配相应的角色

``` 
POST /_security/user/geektime
{
  "password" : "123456",
  "roles" : [ "geektime_role" ]
}
```

配置kibana，启动kibana.

配置刚刚之前创建的kibana这个内置的用户和密码。

验证删除测试一下：

``` 
使用刚创建的用户登录kibana
#会提示失败
DELETE orders
#以下，提示成功
GET orders/_search
PUT orders/_bulk
{"index":{"_id":3}}
{"name":"3333"}
{"index":{"_id":4}}
{"name":"4444"}
```

问题：问下已经通过的球友，确认下，是否就需要配置两个参数？

是否可以在kibana页面中来配置RBAC？



### 配置3节点的集群，同时满足以下要求

* 确保索引A的分片全部落在节点1
* 索引B分片全部落在节点2和3
* 不允许删除数据的情况下，保证集群状态为Greeen

答案如下：

由于查看到官方文档中，\_name是一个内置的属性值，只需要直接设置就可以了。

``` 
PUT test_a
{
  "settings":{
    "index.routing.allocation.include._name": "node1",
    "index.number_of_replicas": 0
  }

}
PUT test_b
{
  "settings":{
     "index.routing.allocation.require._name": "node2,node3"   
  }
}
```

导入一些测试数据：

``` 
POST a_index/_bulk
{"index":{"_id":1}}
{"name":"1111"}
{"index":{"_id":2}}
{"name":"2222"}

POST b_index/_bulk
{"index":{"_id":111}}
{"name":"33333"}
{"index":{"_id":2222}}
{"name":"4444"}
```

最后查看节点的属性，和索引分片的情况：

``` 
GET _cat/nodeattrs
GET _cat/shards?v
```



## 索引数据篇

### 为一个索引，按要求设置以下dynamic Mapping

* 一切text类型的字段，类型全部映射为keyword
* 一切以int_开头命名的字段，类型都设置成integer

题目没有要求索引以什么样的名字为匹配规则，来匹配这里的要求，我们只需要设定一个dynamic template就可以了。

``` 
DELETE index0008
PUT index0008
{
  "mappings": {
    "dynamic_templates": [
      {
        "longs_as_strings": {
          "match":   "int_*",
          "mapping": {
            "type": "integer"
          }
        }
      },
      {
        "longs_as_strings": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
导入测试数据
POST index0008/_bulk
{"index":{"_id":1}}
{"cont":"你好我好铭毅天下好", "int_value":35}
{"index":{"_id":2}}
{"cont":"铭毅天下", "int_value":35}
{"index":{"_id":3}}
{"cont":"铭毅好", "int_value":35}
查看mapping信息
GET index0008/_mapping
```



### 设置一个Index Template，符合以下的要求

* 为log和log-开头的索引。索引3个主分片，1个副本分片。
* 同时为索引创建一个相应的alias.
* 使用bulk API，写入多条电影数据.

``` 
PUT _template/template_001
{
  "index_patterns": ["log*", "log-*"],
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas":1
  },

"aliases" : {
        "alias_001" : {}}
}
```

使用bulk API导入数据

``` 
POST log-001/_bulk
{"index":{"_id":1}}
{"name":"movice_001"}
{"index":{"_id":2}}
{"name":"movice_002"}
{"index":{"_id":3}}
{"name":"movice_003"}
```

验证查询信息：

``` 
GET log-001
GET alias_001/_search
```



### 为movies index设定一个index alias，默认查询只返回评分大于3的电影

先创建一个索引

``` 
PUT movies_index
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "score": {
        "type": "float"
      }
    }
  }
}
```

导入索引必要的数据

``` 
PUT movies_index/_bulk
{"index":{"_id":1}}
{"name":"001","score":1}
{"index":{"_id":2}}
{"name":"001","score":4}
{"index":{"_id":3}}
{"name":"001","score":3}
{"index":{"_id":4}}
{"name":"001","score":3.8}
```

设置aliases

``` 
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "movies_index",
        "alias": "movies_alias",
        "filter": {
          "range": {
              "score":{
                "gt":3
              }
          }
        }
      }
    }
  ]
}
```

使用别名进行查询。

``` 
GET movies_alias/_search
```



### 给一个索引A，要求创建索引B，通过Reindex API，将索引A中的文档写入索引B，同时满足以下要求

* 增加一个整形字段，将索引A中的一个字段的字符串长度，计算后写入
* 将A文档中的字符串以";"分隔后，写入索引B中的数组字段中

这个题目考察的就是reindex+ingest pipeline, 新增字段可以用set processor或者直接使用script processor，分隔字段可以使用split processor。

先定义一个index_a

```
PUT index_a
{
  "mappings": {
    "properties": {
      "title":{
        "type":"keyword"
      }
    }
  }
}
```

插入一些测试数据：

``` 
POST index_a/_bulk
{"index":{"_id":1}}
{"title":"foo bar"}
```

创建ingest pipeline，里面使用script processor和split processor.

``` 
PUT _ingest/pipeline/pipeline-1
{
  "description": "this ia a pipleine-1",
  "processors":[
      {
        "script":{
          "lang":"painless",
          "source":"""
           ctx.length = ctx.title.length()
          """
        }
      },
      {
        "split":{
          "field":"title",
          "separator":";",
          "target_field":"field_inb"
        }
      }
    ]
}
```

执行reindex API

``` 
POST _reindex
{
  "source":{
    "index":"index_a"
  },
  "dest":{
    "index":"index_b",
    "pipeline":"pipeline-1"
  }
}
```

查询验证：

``` 
GET index_b/_mapping
```



### 定义一个pipeline,并且将eathquakes索引的文档进行更新

* pipeline的ID为eathquakes_pipeline
* 将magnitude_type的字段值改为大写
* 如果文档不包含"batch_number"，增加这个字段，将数值设置为1
* 如果已经包含batch_number，字段值加1.

先设置ingest pipeline

``` 
PUT _ingest/pipeline/earthquakes_pipeline
{
  "description": "earthquakes_pipeline",
  "processors": [
    {
      "uppercase": {
        "field": "magnitude"
      }
    },
    {
      "script": {
        "lang": "painless",
        "source": """
              if (ctx.containsKey("batch_number") == true) {
                ctx.batch_number += 1;
              } else {
                ctx.batch_number = 1;
              }
"""
      }
    }
  ]
}

```

做些测试数据

``` 
POST earthquakes/_doc/1
{
  "cont":"1111",
  "magnitude":"asdf"
}
```

最后做一次update_by_query

``` 
POST earthquakes/_update_by_query?pipeline=earthquakes_pipeline
{
  "query":{
    "match_all":{}
  }
}
```



### 为索引中的文档增加一个新的字段，字段值为现有字段1+现有字段2+现在字段3

ingest pipeline中使用script processor，使用ctx.xxxx实现新增字段，和取已有字段的值。

最后使用update_by_query+pipeline来实现所有数据的更新。

```
DELETE test_index
PUT test_index
{
  "mappings":{
    "properties":{
      "value01":{"type":"integer"},
      "value02":{"type":"integer"},
      "value03":{"type":"integer"}
    }
  }
}
```

添加测试数据

``` 
POST test_index/_bulk
{"index":{"_id":1}}
{"value01":1,"value02":2,"value03":3}
{"index":{"_id":2}}
{"value01":3,"value02":2,"value03":3}
{"index":{"_id":3}}
{"value01":5,"value02":2,"value03":3}
```

设置ingest pipeline.

``` 
PUT _ingest/pipeline/newadd_pipeline
{
  "processors": [
    {
      "script": {
        "lang": "painless",
        "source": "ctx.newadd = (ctx.value01 + ctx.value02 + ctx.value03)"
      }
    }
  ]
}
```

执行_update_by_query + pipeline

``` 
POST test_index/_update_by_query?pipeline=newadd_pipeline
POST test_index/_search
```



## 查询篇

### 写一个查询，要求某个关键字在文档的4个字段中至少包含两个以上

* bool查询，should/minimun_should_match

``` 
POST search_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match_phrase": {
            "cont1": "chuck"
          }
        },
        {
          "match_phrase": {
            "cont2": "chuck"
          }
        },
        {
          "match_phrase": {
            "cont3": "chuck"
          }
        },
        {
          "match_phrase": {
            "cont4": "chuck"
          }
        }
      ],
      "minimum_should_match": 2
    }
  }
}
```



### 按照要求写一个search template

* 写入search template
* 根据search template写出相应的query

``` 
POST _scripts/with_reponse_and_tag
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "match": {
          "{{my_field}}": "{{my_value}}"
        }
      },
      "size": "{{my_size}}"
    }
  }
}
```

通过ID来调用

```
POST kibana_sample_data_flights/_search/template
{
  "id":"with_reponse_and_tag",
  "params":{
    "my_field": "DestCityName",
    "my_value":"Sydney",
    "my_size": 10
  }
}
```

可以查看render的效果：

``` 
GET _render/template
{
  "id":"with_reponse_and_tag",
  "params":{
    "my_field": "DestCityName",
    "my_value":"Sydney",
    "my_size": 10
  }  
}
```



### 对一个文档的多个字段进行查询，要求最终的算分是几个字段上的总和，同时要求对特定字段设置boosting值

``` 
POST search_index/_search
{
  "query":{
    "multi_match":{
      "type":"most_fields",
      "query":"chuck",
      "fields": ["cont2^3", "cont3"]
    }
  }
}
```



### 针对一个索引进行查询，当索引的文档中存在对象数组时，会搜索到了不期望的数据。需要重新定义mapping，并提供改写后的query语句

* Nested Object

先定义一个nested对象类型。

``` 
PUT my_index
{
  "mappings": {
    "properties": {
      "user": {
        "type": "nested" 
      }
    }
  }
}
```

写入索引数据。

``` 
PUT my_index/_doc/1
{
  "group" : "fans",
  "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}
```

使用nested的方式对该数据类型，进行查询。

``` 
GET my_index/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "Alice" }},
            { "match": { "user.last":  "Smith" }} 
          ]
        }
      }
    }
  }
}
```

嵌入高亮：

``` 
GET my_index/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "Alice" }},
            { "match": { "user.last":  "White" }} 
          ]
        }
      },
      "inner_hits": { 
        "highlight": {
          "fields": {
            "user.first": {}
          }
        }
      }
    }
  }
}
```



## 聚合篇

### earthquakes索引中包含了过去11个月的地震信息，请通过一句查询，获取以下信息

* 过去11个月，每个月的平均地震等级(magiitude)
* 过去11个月里，平均地震等级最高的一个月及其平均地震等级
* 搜索不能返回任何文档

``` 
POST /_search
{
  "size":0,
  "aggs": {
    "earthquake_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "earthquake_per_month": {
          "avg": {
            "field": "magiitude"
          }
        }
      }
    },
    "max_monthly_magiitude": {
      "max_bucket": {
        "buckets_path": "earthquake_per_month>earthquake_per_month"
      }
    }
  }
}
```



### Query Filter Bucket Filter

每个filter就是一个桶了。

``` 
GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : [
          { "match" : { "body" : "error"   }},
          { "match" : { "body" : "warning" }}
        ]
      }
    }
  }
}
```



### Pipeline Aggregation -> Bucket Filter



## 映射与分词篇

### 一篇文档，字段内容包含了"hello&world",索引后，要求使用match_phrase query

* 查询hello&world或者hello and world都能匹配到。

和dog&cat类似，通过将character_filter将&装换为and。

``` 
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "my_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            "& => and"
          ]
        }
      }
    }
  },
  "mappings":{
    "properties": {
      "title":{
        "type":"text",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```



### reindex索引，同时确保给定的两个查询，都能搜索到相关的文档，并且文档的算分是一样的

* match查询，分别查"smith's", "smiths"
* 在不改变字段的属性，将数据索引到新的索引上
* 确保两个查询有一致的搜索结果和算法

``` 
PUT _template/my_index_template
{
  "index_patterns": [
    "a_index_*"
  ],
  "settings": {
    "number_of_shards": 1,
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "char_filter": [
            "emoticons"
          ],
          "tokenizer": "standard"
        }
      },
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [
            "' => "
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}
```

索引数据

``` 
PUT a_index_001/_bulk
{"index":{"_id":1}}
{"title":"smith's"}
{"index":{"_id":2}}
{"title":"smiths"}

POST a_index_001/_analyze
{
  "text": "smith's",
  "analyzer": "my_custom_analyzer"
}
```

Reindex索引

``` 
POST _reindex
{
  "source": {
    "index": "a_index_001"
  },
  "dest": {
    "index": "a_index_002"
  }
}
```

测试

``` 
POST a_index_002/_search
{
  "query": {
    "match_phrase": {
      "title": "smith's"
    }
  }
}
```



## 集群管理篇

### 安装并配置一个hot&warm架构的集群

* 三个节点，node1为hot，node2为warm，node3为cold
* 三个节点均为master-eligable节点
* 新创建的索引，数据写入hot节点
* 通过一条命令，将数据从hot节点迁移到warm节点

首先node1和node2上设置不同的属性值。

``` 
node1的elasticsearch.yml添加如下：
node.attr.hot_warm_type: hot

node2的elasticsearch.yml中添加如下：
node.attr.hot_warm_type: warm
```

设置索引的setting值。

``` 
PUT test/_settings
{
  "index.routing.allocation.include.hot_warm_type": "hot",
  "number_of_shards": 1,
  "number_of_replicas": 0
}
```

索引一些文档数据。

``` 
PUT test/_bulk
{"index":{"_id":1}}
{"name":"elastic.blog.csdn.net"}
{"index":{"_id":2}}
{"name":"铭毅天下"}
```

查看分片分配信息：

``` 
GET _cat/shards?v
```

修改索引的setting参数的值。

``` 
PUT test/_settings
{
  "index.routing.allocation.include.hot_warm_type": "warm"
}
```

再次查看分片热迁移的情况：

``` 
GET _cat/shards?v
```



### 为两个集群配置跨集群搜索

* 两个集群都有movies的索引
* 创建跨集群搜索
* 创建一条

首先在集群的API上进行设置。

``` 
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster_one": {
          "seeds": [
            "127.0.0.1:9300"
          ]
        },
        "cluster_two": {
          "seeds": [
            "127.0.0.1:9301"
          ]
        }
      }
    }
  }
}
```

进行跨集群的检索

``` 
GET /orders,cluster_two:orders/_search
{
  "query": {
    "match_all": {}
  }
}
```



### 解决集群变红或者变黄的问题

* 技能1：通过explain API查看
* 技能2：shard filtering API，查看include

``` 
查看集群状态
GET _cluster/health
返回状态举例："status":"red"，红色，至少一个主分片未分配成功。

到底哪个节点出现了红色或黄色问题呢？
GET _cluster/health?level=indices

如下的方式，更明快直接找到对应的索引
GET _cat/indices?v&health=yellow
GET _cat/indices?v&health=red

到底索引的哪个分片出现了红色或黄色的问题呢?
GET _cluster/health?level=shards

到底什么原因导致了集群变成红色或黄色呢？
GET _cluster/allocation/explain
```

* 技能3：更新一下routing，确认replica可以分配(include更加多的rack)

解决方案：

```
1) 为集群配置延迟分配和一个节点上最多几个分片的配置
2) 设置Replica为0
3) 删除dangling index
4) 使用了错误的routing node attribute
```



### 备份一个集群中指定的几个索引

首先在elasticsearch.yml中定义一个共享目录，也就是镜像目录。

``` 
path.repo: ["/home/elasticsearch/elasticsearch-7.2.0/backup"]
```

重启节点，随后，注册一个snapshot的仓库

``` 
PUT /_snapshot/my_fs_backup
{
    "type": "fs",
    "settings": {
        "location": "/home/elasticsearch/elasticsearch-7.2.0/backup/snapshot",
        "compress": true
    }
}
```

查看索引的情况

``` 
GET _cat/indices
```

在注册的仓库中，创建一个snapshot

``` 
PUT /_snapshot/my_backup/snapshot_2?wait_for_completion=true
{
  "indices": "orders",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

查看创建的snapshot

``` 
GET  /_snapshot/my_backup/snapshot_2
```

随后删除索引

``` 
DELETE orders
```

使用snapshot进行恢复

``` 
POST /_snapshot/my_backup/snapshot_2/_restore
```

最后查询验证

``` 
GET orders/_search
```



# 真题训练三

## 部署实战

题目摘录：

The hosts node1,node2, and node3 are configured to communiate with each. These three nodes are currently not running, but a kibana instance is running on port 5601 and configured to connect to the cluster.

Deploy an Elasticsearch cluster that satisfies the following requirements:

* node1 is a dedicated master node and the only master-eligible node in the cluster
* node2 and node3 are data and ingest nodes only
* the name of the cluster is cluster1
* the node names match the server names of each node

答案如下：

node1的配置

``` 
cluster.name:  cluster1
node.name: node1
network.host: 10.128.190.86
discovery.seed_hosts: ["node1","node2","node3"]
cluster.initial_master_nodes: ["node1"]
node.master: true
node.data: false
node.ingest: false
node.ml: false
cluster.remote.connect: false 
```

node2的配置

``` 
cluster.name: cluster1
node.name: node2
network.host: 10.128.190.87
discovery.seed_hosts: ["node1","node2","node3"]
cluster.initial_master_nodes: ["node1"]
node.master: false
node.data: true
node.ingest: true
node.ml: false
cluster.remote.connect: false 
```

node3的配置

``` 
cluster.name: cluster1
node.name: node3
network.host: 10.128.190.88
discovery.seed_hosts: ["node1","node2","node3"]
cluster.initial_master_nodes: ["node1"]
node.master: false
node.data: true
node.ingest: true
node.ml: false
cluster.remote.connect: false
```



## Mapping定义实战题

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

* The name of the index is task2
* The username field is mapped as a keyword only
* The field in address are all mapped as both text and keyword
* The comment field is indexed as text three different ways: using the standard analyzer, the english analyzer, and the dutch analyzer

Index the document above into your new task2 index with an _id of 123.

答案如下：

设置mapping信息

```
PUT task2
{
  "mappings": {
    "properties": {
      "username": {
        "type": "keyword"
      },
      "address": {
        "properties": {
          "city": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          },
          "state": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          },
          "country": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          }
        }
      },
      "comment": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "english": {
            "type": "text",
            "analyzer": "english"
          },
          "dutch": {
            "type": "text",
            "analyzer": "dutch"
          }
        }
      }
    }
  }
}
```

index 文档

```
POST task2/_doc/123
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

查看mapping信息

``` 
GET task2/_mapping
```



## 查询实战题

There is a Kibana instance configured for cluster2 that is running on port 5602. Write a single search on the movie_data index on cluster2 that satisfies the following requirements:

* The tags field containes "based on comic book" and "marvel cinematic universe"
* The results are sorted first by budget descending, then release_data descending

In the text field below, provide only the JSON portion of your search request:

``` 
POST movie_data/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "match": {
            "tags": "based on comic book"
          }
        },
        {
          "match": {
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



# 真题训练四

一个文档中的字段是一个数组类型，定义一个ingest pipeline，使用foreach processor提取出数组中的每一个数组元素对象，对该每个数组对象使用trim processor，删除每个数组对象中的开头的空格和结尾的空格。

先定义一个存在数组元素的对象的字段。

``` 
POST test_005/_bulk
{"index":{"_id":1}}
{"tags":["ping pang", "basket ball", " foot bool "]}
{"index":{"_id":2}}
{"tags":[" ping pang ", "gof bal"]}
```

设置一个ingest pipeline，foreach processor提取出每个数组元素，然后再用trim proccessor修剪开头的空格和结尾的空格。

``` 
PUT _ingest/pipeline/trim_pipeline
{
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

设置一个新的索引，reindex+pipeline

``` 
DELETE task6
PUT task6
POST _reindex
{
  "source": {"index":"test_005"},
  "dest":{"index":"task6", "pipeline": "trim_pipeline"}
}
GET task6/_search
```



# 球友真题整理

考前需要对这些真题反复练习，把星球中通过的同学整理的总结反馈看好几遍。按照真题来，反复手敲代码。尝试做到能够手敲出来，对于一些自定义分词这种复杂的DSL，也要做到复制出了DSL之后，能够自己改动一遍以后，基本上是没有错误的。

分析下，哪些是高频题目，哪些是低频题目。

FFFro的复习计划：

6.25 ~ 7.10  每天把以前做过的题再做一道，并且仔细阅读相关文档，保证这道题可以非常准确的找到文档并且很快的做出来。 

7.11 ~ 7.20  复习一遍考试内容，查缺补漏，做一些球友友分享的真题。 

7.21 ~ 7.30  整理下考试题，模拟几次考试，再看下有什么不会的。 

8.1 ~   找时间考试 之前学习都学的太散了，这次专门针对下学习。



群主的刻意练习的建议：

第一：刷题，力扣排行或者各种排行榜，排在前列找工作很好找。

第二：封装属于自己的代码库。

我见过身边的大牛，写代码非常快。深入了解知道：他把之前所有遇到的库、常用接口、函数都封装成自己的库了，比如：字符串处理库、各种封装接口，这样所有项目或者产品开发，就基于已有封装的东西，自然会很快，至少比大多数人快。

第三：所有的方法都可以刻意练习习得。

包括：认证考试，包括敲代码、包括算法等一切底层原理。

测试测试



## 题型出现统计表

| 题型介绍                                       | 备注                                                         | 出现次数 |
| ---------------------------------------------- | ------------------------------------------------------------ | -------- |
| index shard allocation                         | 索引分片重新分配(两个索引一个要固定到1个节点，一个要固定到两个节点)，1个节点的需要修改replicas为0<br>将索引A所有shards分配到node1，索引B的所有shards分配到node2+node3 (index allocation) <br>hot/warm集群架构(相当于索引级别分片分配，官方叫做routing) | 6        |
| shard awareness                                |                                                              | 8        |
| shard allocation awareness/foced awareness配置 | 对集群进行shard allocation awareness和forced awareness配置，一共4个node，分为两个rack，rack-A和rack-B | 2        |
| oz同义分词，141个结果                          | 自定义analyzer<br>reindex+自定义分词，确保对两个不同的term，返回的评分和结果是一致的。重点是char_fitler和自定义mapping。<br>同义词oz/Oz/OZ同义词转化为occuers。重点是配置standard的tokenizer和lowerupper的token filter。<br>我的做法和他有一点区别，就是在synonym filter之前我还加了一个lowercase filter。然后同义词只设置oz => ounces。<br>给定一个索引index（包含数据）和两个match_phrases查询（两个查询结果各多少条，133和8），reindex 到一个新索引使得包含oa,oA,OA,onse的phrases查询结果都是144条记录。 | 9        |
| ingest pipeline去掉'后reindex                  | 先自定义分词，再reindex，保证kings和king's评分一样。这个题目可以通过自定义char_filter处理，群主的github有对应的真题，如果是为了省时间，可以直接使用english分析器。<br>正确的排除' 对查询的影响，确保对waynes/wayne's , king/king's 的查询有一样的结果和分值(custom analyzer) | 9        |
| pipeline+reindex                               | foreach，trim。script (三个字段合并为一个新字段) <br>reindex+pipeline,foreach+trim，set的连接 | 5        |
| pipeline+update_by_query                       | 多个字段的拼接<br>为索引添加一个新字段e是已有四个字段a b c d的拼接(pipeline + update_by_query)<br>新增一个filed，然后需要由"A B C D"四个field按顺序组成，用到set processor和 _update_by_query | 6        |
| 给索引加字段，是多个字段的拼接                 | pipeline(script)+update_by_query                             | 2        |
| pipeline script params                         |                                                              | 1        |



##  球友胜东分享

一共十道题。



### Task1

索引分片重新分配



### Task2

索引reindex，分词的时候去掉单引号



### Task3

给索引加字段，是多个字段的拼接



### Task4

dynamic template 



### Task5

聚合查询



### Task6

创建nested对象，进行搜索



### Task7

地震数据聚合



### Task8

RBAC



### Task9

boost + highlilght + sort



### Task10

跨集群搜索



## 球友思炀的分享

考试10道题目，共计4个集群。



### Task1

shard awareness



### Task2

oz同义分词， 141个结果



### Task3

search + highlight + sort ， 50几个结果



### Task4

Cross cluster search



### Task5

multi_field多种分词器设置



### Task6

template mapping



### Task7

RBAC



### Task8

function score



### Task9

filter + aggs



### Task10

reindex + pipeline



## 球友IBM周钰分享

10道大题，3个小时。



### Task1

shard allocation filter配置，ES集群冷热配置，最终目标是确保分片在规定的节点上，并保证集群是Green。



### Task2

启用ES的安全配置的功能，修改内置账号的密码，创建一个第三方的用户，并且为该用户赋权。



### Task3

聚合，找出每个月的最大震级和深度。



### Task4

一个布尔查询，要求高亮查询结果，并且对指定关键字进行排序。主要match phrase.



### Task5

一个多字段查询，返回的结果是各个字段的总和。注意boost



### Task6

针对特定索引的snapshot



### Task7

pipeline + update_by_query，对某个索引增加一个字段，该字段的内容是其他3个字段的连接。



### Task8

Reindex + 自定义分词，确保对两个不同的term，返回的评分和结果数量是一致的。重点是char_filter，和自定义mapping。



### Task9

Dynamic mapping template



## 球友shane分享

### Task1 

awareness



### Task2

fileds定义多个字段，并且每个字段定义自己的analyzer(standard, english, stop) + 基本reindex



### Task3

2个集群跨集群搜索配置 + 在cluster2中搜索movies-1，在cluster3中搜索movies-2



### Task4

同义词oz，Oz,OZ 同义转化为occuers 。重点是配置standard的tokenizer和lowcupper的token filter。



### Task5

function score搜索my, me， 当tags存在romatic时，评分上升。



### Task6

reindex+pipeline， foreach，trim (数组中每一个value都去掉空白的前缀和后缀)，script(三个字段合并一个新字段)



### Task7

bool (should，match_phrase)，hightlight(高亮搜索到的name字段的值，前缀为\<b>,后缀为\</b>)， sort (注意要使用field.keyword)



### Task8

静态模板aaa开头的必须有特定的mapping，某个字段为text，某个为keyword，某个为date。要注意诸侯要插入一个id为1的文档。



### Task9

RBAC elastic和kibana的密码都是为password，添加一个jian的用户，fullname为Jian，email为aaa@aa.com, 指定kibana_user



### Task10

先query搜索出指定term的文档，再terms聚合出前10的文档。



## 球友金多安的分享

### Task1

shard awareness



### Task2

reindex+pipeline， foreach+trim , set的连接



### Task3

oz的同义词分词器    141个结果



### Task4

CCS



### Task5

search + highlight + sort



### Task6

filter + aggs (食物的那题)



### Task7

multi_field的不同分词器设置



### Task8

带对象字段的template



### Task9

security + RBAC (单节点+jian)



### Task10

function score



## 球友Frank的分享

reindex如果遇到复杂的场景，必须要使用脚本处理时，我建议还是ingest去包裹脚本，而不是直接使用脚本更新。

ingest相比较直接使用脚本最大的好处，就是便于测试，如果题目没有特别说明，必须使用脚本，建议使用ingest去包裹脚本。

在做每道题的时候，最好看一下索引的mapping是什么样的。



### Task1

sharding awareness，保持cluster健康



### Task2

先自定义分词，再reindex，保证kings和king's评分一样。这个题目可以通过自定义char_filter处理，群主的github有对应的真题，如果是为了省时间，可以直接使用english分析器。



### Task3

先nested字段定义，再reindex，最后nested查询。

这题稍微费了一些时间，需要重新定义mapping，再reindex，最后写查询语句。



### Task4

multi_match和most_filed



### Task5

RBAC(单节点)

这个题有两个要求，enable security和create user

只需要开启xpack，不需要去配置证书，还有在命令行交互模式下，输入密码时要慎重ctr+v，直接用手敲。



### Task6

update_by_query，多个字段拼接。

更新字段比较简单，不用写脚本。



### Task7

先date_histogram，再max depth



### Task8

生成快照



### Task9

dynamic mapping，按字段类型和前缀做动态映射。



### Task10

query，higthlight和sort



## 球友Sun的分享

共有4套不同的ES集群，对应不同的试题，其中一套环境涉及到多道题目。



### Task1

对集群进行shard allocation awareness和forced awareness配置，一共4个node，分为两个rack，rack-A和rack-B



### Task2

针对食品供应商的index，进行同义词搜索，通过自定义的analyzer完成。



### Task3

简单search涉及到bool，highlight，sort，size，phrase等知识点。



### Task4

简单phrase search，对包含某phrase的文档提分。



### Task5

跨集群搜索



### Task6

创建一个index template



### Task7

ES安全设置和角色设置，可通过kibana界面快速完成。



### Task8

针对index中的某一字段，实现muliti fields.



### Task9

ingest pipeline + script + reindex



### Task10

针对食品供应商index，满足name中包含某字段，然后返回全部供应商生产的top10的商品。



## 球友袁麟的分享

考试的时候，我检查了一下环境，发现是单机上跑的docker，8个容器，3个集群。

* cluster1: node1/node2/node3/kibana1
* cluster2: datanode1/kibana2
* cluster3: prod1/kibana3

kibana都启动了，ES都没有启动，自己使用

```/home/elastic/elasticsearch/bin/elasticsearch -d -p pid```启动起来

考试总共考了10道题目。



### Task1

将索引A所有shards分配到node1，索引B的所有shards分配到node2+node3 (index allocation)



### Task2

正确的排除' 对查询的影响，确保对waynes/wayne's , king/king's 的查询有一样的结果和分值(custom analyzer)



### Task3

要求将string类型映射为keyword， 将x_开头的字段映射为integer (dynamic template)



### Task4

为索引添加一个新字段e是已有四个字段a b c d的拼接(pipeline + update_by_query)



### Task5

对三个字段a/b/c 查询xxx，要求c字段boost 2，各字段查询算分加和(bool query should 或者multi_match)



### Task6

将a字段定义为nested，将原始idx索引reindex到新索引，确认对nested object查询的结果(nested object + nested query)



### Task7

earthquakes索引按照月份进行聚合，并统计出每月magnitude和deth最大值 (date_histogram aggregation + max aggregation)



### Task8

在路径/home/elastic/backups建立快照目录myrepo，对索引movie创建快照movie_1 (snapshot)



### Task9

启用security，配置elastic/kibana密码为password，创建用户susan，密码为password，邮件certificate@elatic.co，角色kibana_user (RBAC)



### Task10

对movie的title使用match phrase查询star trek，按照revenue倒序排序，title字段用\<strong>  \</strong> 高亮 (order+highlight)



## 球友杜亚朔的分享

整理不会做或是不确定的习题，对掌握不好的知识点重点突破，看文档，教程，查资料都可以。

重新做一遍习题。做到最后，官方文档和阮老师的课程也不知道反反复复的看了多少边。基本上䏻做到写常用的DSL和创建索引可以不用看文档了（依赖于Kibana的提示功能很好）。

为了节约时间，做创建指定索引类型的题目时，可以随便创建个索引然后获取索引的信息，再按照题目要求修改下。

由于我的VPN不给力，考试环境里面的浏览器切换tab有时都会卡，我都会把题目拷贝到kibana上面。

考试使用的邮箱之前注测过，交费的时候一直提示有错，后来换了个邮箱（看到之前有位朋友也遇到了一样的问题）



### Task1

集群allocation-awareness配置



### Task2

自定义分词让搜索结果为指定的条数



### Task3

pipeline + reindex



### Task4

pipeline + update_by_query



### Task5

按照要求配置索引 + reindex



### Task6

聚合，貌似是每个月聚合下



### Task7

查询，高亮，排序，指定返回数量



### Task8

index templates



### Task9

创建用户，给kibana_user权限



## 球友红烧小白兔的分享

考试时候桌面的那个官方文档是一个大索引，之前查文档我从来没有这么玩过，找到正确的文档也花了好一会儿，这个是我比较失误的一个地方，如果能提前熟悉，至少能节省一些时间。考试的集群有4个，考试界面有整整一页都是描述这个集群，要仔细看一看。



### Task1

对集群进行allocation-awareness配置



### Task2

跨集群搜索



### Task3

之前很多群友见过的食品供应商索引的同义词Analyzer搜索配置



### Task4

一个bool search



### Task5

function_score

``` 
索引 movie-1，保存的电影信息，title是题目，tags是电影的标签。

要求：

1. 在title中包含“my”或者“me”。

2. 如果在tags中包含"romatic movies"，该条算分提高，如果不包含则算分不变
```

球友分享的答案如下：

``` 
GET /movie-1/_search
{
  "query": {
    "function_score": {
      "query": {
        "bool": {
          "should": [
            {
              "match": {
                "title": "my"
              }
            },
            {
              "match": {
                "title": "me"
              }
            }
          ]
        }
      },
      "boost": "5",
      "functions": [
        {
          "filter": {
            "term": {
              "tags.keyword": "romatic movies"
            }
          },
          "weight": 5
        }
      ]
    }
  }
}
```

这样做的好处是后面的term search只对评分有影响，对搜索结果是没有影响的。

不过这个结果我没有验证过，我的ES机器之前已经关掉了，等后面我验证过会来更新这个结果。



### Task6

ES安全设置和角色配置



### Task7

动态mapping



### Task8

ingest pipeline + script + reindex



### Task9

还是食品供应商那道题，满足name中包含某字段，然后返回商品top10的供应商的名字。

一个就算同义词那题，很多群友反馈题目错了，我做完后搜索出来的总条目是141和题目中要求的是一致的，不是群友帖子里面的144，也许后来官方调整过了。我的做法和他有一点区别，就是在synonym filter之前我还加了一个lowercase filter。然后同义词只设置oz => ounces。



## 球友阿炳的分享

尽量不要用笔记本考试，有大屏用大屏。

屏小字小还模糊，没考完眼睛就瞎了。

平时多多习惯英文阅读，考试不慌。与星友碰到同一个问题，我们都坚信自己的语句没错，却得不到官方想要的结果。这时，你可能会怀疑自己。



### Task1

给定一个索引index（包含数据）和两个match_phrases查询（两个查询结果各多少条，133和8），reindex 到一个新索引使得包含oa,oA,OA,onse的phrases查询结果都是144条记录。

查询语句：

``` 
GET my_index/_search
{
  "query": {
    "match_phrase": {
      "title": "8 oa"
    }
  }
}
```

下面是球友的考试中的第一思路（由于得不到144的结果，考试中这题被我改得面目全非，错误是肯定的

``` 
DELETE my_index*
#定义同义词分析器
PUT my_index
{
  "settings": {
    "index": {
      "analysis": {
        "analyzer": {
          "my_custom_analyzer": {
            "tokenizer": "standard",
            "filter": [
              "synonym"
            ]
          }
        },
        "filter": {
          "synonym": {
            "type": "synonym",
            "lenient": true,
            "synonyms": [
              "oa,oA,OA,onse"
            ]
          }
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}
```

索引原始数据

```
PUT my_index/_bulk
{"index":{"_id":1}}
{"title":"hello 8 oa world"}
{"index":{"_id":2}}
{"title":"hello 8 oA world"}
{"index":{"_id":3}}
{"title":"hello 8 OA world"}
{"index":{"_id":4}}
{"title":"hello 8 onse world"}
```

验证分析器

```
GET my_index/_analyze
{
  "text": "8 oa",
  "analyzer": "my_custom_analyzer"
}
GET my_index/_search
{
  "query": {
    "match_phrase": {
      "title": "8 oa"
    }
  }
}
```



### Task2

4个节点，配置awareness, force



### Task3

创建用户jian，给kibana_user权限



### Task4

query bool，包含should，must，must_not



### Task5

pipeline script params



### Task6

top_hits aggregation



## 球友skillip的分享

我觉得Context examples、Ingest processor context、Update context、Update by query context、Reindex context、Score context、Filter context是必看的，对应的ES文档结合起来看。

由于ctx是Map，我们还必须能正确拼写Map的常用方法，Kibana毕竟不能像IDE那样有提示。如，containsKey（多了s）, containsValue,还有像size，isEmpty等一般不容易拼错。 写Painless时，尽量使用三重双引号，即"""，这样里面的代码就不需要转义了。

分享桌面的时候出了些小状况，它会弹出一个对话框，但是此时分享按钮是灰色，需要点击那个图片，分享按钮才会变成蓝色，此时才可以点击。我当时申请刷新了两次页面，以为浏览器出了Bug，也增加了不必要的紧张情绪。

如果你的题目是关于食品添加剂（food ingredient）的，那么有道题可能很难看懂，该题目就一句话，但是写了三排，英语不太好，没看懂，要返回前10制造商的items？但是没有这个字段，括号里的解释有这样manufacturer's of的语法结构，花了20分钟，没能理解。



### Task1

集群的配置



### Task2

RBAC的配置



### Task3

同义词转换



### Task4

高亮的前缀和后缀



### Task5

multi-field的配置



### Task6

index template



### Task7

ingest



### Task8

跨集群搜索



## 球友Desmond的分享

ES官方的考试内容大纲：

``` 
Installation and Configuration

1. Deploy and start an Elasticsearch cluster that satisfies a given set of requirements
2. Configure the nodes of a cluster to satisfy a given set of requirements
3. Secure a cluster using Elasticsearch Security
4. Define role-based access control using Elasticsearch Security

Indexing Data
1. Define an index that satisfies a given set of requirements
2. Perform index, create, read, update, and delete operations on the  documents of an index
3. Define and use index aliases
4. Define and use an index template for a given pattern that satisfies a given set of requirements
5. Define and use a dynamic template that satisfies a given set of requirements
6. Use the Reindex API and Update By Query API to reindex and/or update documents
7. Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents

Queries
1. Write and execute a search query for terms and/or phrases in one or more fields of an index
2. Write and execute a search query that is a Boolean combination of multiple queries and filters
3. Highlight the search terms in the response of a query
4. Sort the results of a query by a given set of requirements
5. Implement pagination of the results of a search query
6. Use the scroll API to retrieve large numbers of results
7. Apply fuzzy matching to a query
8. Define and use a search template
9. Write and execute a query that searches across multiple clusters

Aggregations
1. Write and execute metric and bucket aggregations
2. Write and execute aggregations that contain sub-aggregations

Mappings and Text Analysis
1. Define a mapping that satisfies a given set of requirements
2. Define and use a custom analyzer that satisfies a given set of requirements
3. Define and use multi-fields with different data types and/or analyzers
4. Configure an index so that it properly maintains the relationships of nested arrays of objects

Cluster Administration
1. Allocate the shards of an index to specific nodes based on a given set of requirements
2. Configure shard allocation awareness and forced awareness for an index
3. Diagnose shard issues and repair a cluster’s health
4. Backup and restore a cluster and/or specific indices
5. Configure a cluster for use with a hot/warm architecture
6. Configure a cluster for cross cluster search
```

考试技巧：

* 有一个问题是：去掉前缀空格和后缀空格。其对应的英文是leading spaces 和 trailing spaces，我花了好长时间猜它的意思，总算没猜错。多了解下Ingest Processor，有些场景其比analyzer更适用。
* 有需要配置集群的问题，启动



## 球友95后运维小哥

还有一个致命问题小伙伴们记下，我用的笔记本，当考试开始后，由于开着麦克风有杂音我就关掉了麦克风，在连接意外断开后要重新启用麦克风浏览器才会调用摄像头，不然点击分享摄像头是没有反应的，这一点切记！！当时和考官沟通了10分钟，最后想到试着打开麦克风解决了，这个可以通过调小音响音量来解决，这只是我遇到的情况，给小伙伴们提个醒。



### Task1

基础security模块和RBAC



### Task2

集群管理把不是green的分片弄成green(改副本数)



### Task3

hot/warm集群架构 (相当于索引级别分片分配，官方叫做routing)



### Task4

实例级别的分片分配(一个elasticsearch 进程一个实例)



### Task5

reindex + script



### Task6

snapshot



### Task7

update_by_query + pipeline



### Task8

ingest



### Task9

template



### Task10

自定义分析(custom analysis，要搞清楚文本分词、文本过滤、字符过滤的功能)



### Task11

mapping根据条件设定数据类型



### Task12

分页查询，highlight(自定义高亮标签)、排序



### Task13

聚合的三种方式



## 球友天小希的分享1

这次考试环境是4个集群，第一个集群是4节点，其他三个都是单节点。



### Task1

集群状态的创建，需要结合attr和awareness



### Task2

reindex+ingest_pipeline，难点是其中有一个字段是数组，数组的内容大概如下，reindex的时候，需要去到这个数组里面的多余的空格（我的理解是第一个空格要去掉？），然后要新增一个field，fullname= xxx + name

``` 
text: {
' abcedf ds s'
' 123abc'
' abc123'
' cde123'
}
```



### Task3

query_search，match_phrase，should，highlight等等



### Task4

bucket aggr，filter某些字段，sort，size=0



### Task5

单节点的RBAC，最后是创建一个jian的用户，关联kibana_user的role



### Task6

synonym，需要match"oz Oz OZ => nuxxxx"



### Task7

dynamic template



### Task8

新建一个index，需要和原有的index有相同的mapping，但是某些字段需要新增filed或者修改类型。



### Task9

跨集群搜索



## 球友天小希的分享2

1. 考试是可以外接显示器的，用大屏的显示器非常有优势。但是考试的centos的分辨率比较低，我尝试过想改分辨率，考官也同意了，但是发现是没有权限改的，所以只能在浏览器上缩小来获取更大的视觉范围。

2. 经过群主的提醒，考试的题目可以复制到kibana上，再对着来敲，这样就不用来回切换了。但是我发现用大屏和网络好了之后，好像也没这个必要了，大家根据具体情况来吧，我觉得如果是笔记本的话，应该挺有用的

3. 启动es的时候可以在es目录下./bin/elasticsearch -d -p pid

   停止进程在es目录下用kill `cat pid`

4. 注意审题，包括题目的每一个要求，尤其是最后框里很多题只需要输入json的 千万别范我一样的错误，把初始密码设置了，回头想改就比较麻烦了。对于运维经验不强的同学来说，困难就比较大了.



### Task1

index allocation，需要用到awareness和force，这题主要是有两个index，分别是3个shares和1个副本，第一个index，需要你把所有的share都落到node1上，第二个index需要把所有share都落到node2和node3上。

ps：这题我一开始挺困惑的，设置了node的attr，然后设置index的require attr，后面发现设置了awareness和force，至少node2和node3的就正常了。然后发现第一个index还是落在1 2 3上，最后把副本数改成0，然后3个shares也都落到node1上了。



### Task2

 dynamic mapping，reindex，需要查询wayne's和waynes有相同的命中和评分



### Task3

新增一个filed，然后需要由"A B C D"四个field按顺序组成，用到set processor和 _update_by_query



### Task4

nested type和nested query



### Task5

dynamic template，所有string字段设置为keyword，如果命中x_*的，就设置为integer



### Task6

 Each query item, catagroy, item_description of filed fire，然后item_description这个filed要增加boost 2，最后算分的时候，需要3个field相加，而不是选最大值。

ps：我不确定bool should是不是最后算分是sum，但是应该没有错



### Task7

earthquare的date_histogram聚合，然后选出magnitude和depth的最大值



### Task8

单机的rbac，创建一个susan的账号，然后分别kibana_user的权限

PS：这题我犯了打错，题目要求设置elastic和kibana的密码都是password，但是我初始化的时候都设置成了elastic，最后发现再也改不了了，所以只能去kibana那里改连接es的密码，连进去之后，重新再把elastic和kibana两个账号的密码改回来。这里最后的瑕疵是kibana进程，他是开启机器的，最后我是kill掉它，手动启动了，不过问题应该不大。



### Task9

单机snapshot



### Task10

match_phrase，highlight，sort



