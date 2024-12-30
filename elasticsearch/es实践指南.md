### 常用查询

#### 查询分词结果

```json
GET _analyze

{
  "analyzer": "ik_smart",
  "text": "全文测试参股222过不长度测试的关键词搜索测试关键词再次长度"
}
```

其中，`analyzer`表示分词器，目前常用的分词器有两个，一个是**ik_smart**, 另一个是**ik_max_word**。

前者会将文本做最细粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为“中华人民共和国、中华人民、中华、华人、人民共和国、人民、共和国、大会堂、大会、会堂等词语。

后者会做最粗粒度的拆分，比如会将“中华人民共和国人民大会堂”拆分为中华人民共和国、人民大会堂。


### 进阶查询

下面这个查询语句可以实现分词匹配，同时针对特定字段做了加权处理(`fields`字段)，通过`analyzer`指定分词器

```json
GET /100101951057547274620933/_search

{

    "query": {
        "bool": {
            "must": {
                "multi_match": {
                    "query": "全文测试参股222过不长度测试的关键词搜索测试关键词再次长度",
                    "analyzer": "ik_smart",
                    "operator": "or",
                    "fields": [
                        "title^3",
                        "tag^2",
                        "summary^2",
                        "text"
                    ],
                    "minimum_should_match": "50%"
                }
            },
            "filter":{
              "term": {
                    "type": 1
                }
            }
        }
    },
    "highlight": {
        "fields": {
            "title": {},
            "text": {},
            "summary": {},
            "tag": {}
        }
    },
    "from": 0,
    "size": 1000
}
```


### 创建索引（高级用法）

通过`analyzer`指定分词器，通过`search_analyzer`指定搜索分词器

```json
PUT /2512-new/
{
 "mappings": {
      "doc": {
        "properties": {
          "cover_url": {
            "type": "keyword"
          },
          "creator_id": {
            "type": "keyword"
          },
          "creator_name": {
            "type": "keyword"
          },
          "enterprise_id": {
            "type": "keyword"
          },
          "id": {
            "type": "keyword"
          },
          "summary": {
            "type": "text",
            "analyzer": "ik_smart"
          },
          "tag": {
            "type": "text"
          },
          "text": {
            "type": "text",
            "analyzer": "ik_smart"
          },
          "title": {
            "type": "text",
            "analyzer": "ik_smart"
          },
          "type": {
            "type": "integer"
          },
          "update_time": {
            "type": "date"
          },
          "url": {
            "type": "keyword"
          }
        }
      }
    },
    "settings": {
            "number_of_replicas": 1,
            "number_of_shards": 1
            }
    }
```

其中`setting`设置的是分片数等配置，需要根据实际机器配置设置

### 将旧数据复制到新索引结构

```json
POST _reindex
{
  "source": {
    "index": "2512-new"
  },
  "dest": {
    "index": "2512-new2"
  }
}
```

### 批量删除索引

```json
// 同时删除多个索引
DELETE /index100101,index100102,index100103
```


### 查询索引setting信息

```json
// 只看某个值
GET /_all/_settings?filter_path=**.creation_date
```

### 查询索引信息

#### 按时间排序

```json
 GET /_cat/indices?v&s=creation.date:desc
```


### 删除数据

```json
DELETE /authority_resource_data/_doc/1455818647728689179_user_1820179489074843648
```