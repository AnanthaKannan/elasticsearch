# Elastic search

### Delete index
```js
DELETE /index_name
```
### Create index
```js
PUT /index_name
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}
```

### Get the mapping details from the index
```js
GET /index_name/_mapping
```

### Add the document to the index
```js
POST /index_name/_doc
{
  "name": "Coffee maker",
  "price": 97,
  "in_stack": 10
}
```

### Update the doc
```js
/* 
  Just consider _id is 100, if the Id is not exist, it will create new data, otherwise it will update the old data

  action.auto_create_index, you should be enable this as true, so you can create the doc if it is not exist
*/ 
PUT /index_name/_doc/100
{
  "name": "Toaster",
  "price": 85,
  "in_stack": 12
}

POST /index_name/_update/100
{
  "doc": {
    "price": 91
  }
}
```

### Get the Doc
```js
GET /index_name/_doc/100
```

### Delete the Doc
```js
DELETE /index_name/_doc/100
```

### Increment the values with out get the data (scripted update)
On the scripting you can add some script to update the data
```js
POST /index_name/_update/100
{
  "script": {
    "source": "ctx._source.price += 1",
    "lang": "painless"
  }
}
```

### Search the Doc with sorting and pagination and filter and highlight
```js
GET /index_name/_search
{
  "query": {
    "match": {
      "name": "Coffee maker"
    }
  },
  "sort": [
    {
      "price": "asc"
    }
  ],
  "from": 0,
  "size": 2,
  "filter": {
    "term": {
      "in_stack": 10
    }
  },
  "highlight": {
    "fields": {
      "name": {}
    }
  }
}
```

### Search the Doc
```js
GET /index_name/_search
{
  "query": {
    "match": {
      "name": "Coffee maker"
    }
  }
}
```

# Aggregation
### Basic aggregation
```js
GET /index_name/_search
{
  "size": 0, 
  "aggs": {
    "total_sales": {
      "sum": {
        "field": "price"
      }
    },
    "average_sales":{
      "avg": {
        "field": "price"
      }
    },
    "min_sales":{
      "min": {
        "field": "price"
      }
    },
    "max_sales":{
      "max": {
        "field": "price"
      }
    }
  }
}

// both query are give the same response
GET /index_name/_search
{
  "size": 0,
  "aggs": {
    "price_status": {
      "stats": {
        "field": "price"
      }
    }
  }
}
```

### Get the distinct values
```js
GET /index_name/_search
{
  "size": 0, 
  "aggs": {
    "distinct_values": {
      "terms": {
        "field": "name.keyword"
      }
    }
  }
}
```

### eturns an array of all unique values that results from applying an expression to each document in a group of documents
```js
GET /save_data/_search
{
  "size": 0,
  "query":{  
      "match": {
      "organization.keyword": "627cf6467099baa51ec44f5b"
      }
    },
  "aggs": {
    "id": {
      "terms": {
        "field": "name.keyword",
        "order": {
          "_key": "desc"
        }
      },
      "aggs": {
        "commandType": {
          "terms": {
            "field": "commandType.keyword"
          }
        },
       "bucket_truncate": {
          "bucket_sort": {
            "from": 0,
            "size": 100
          }
        }
      }
    },
    "totalCount": {
      "cardinality": {
        "field": "name.keyword"
      }
    }
  }
}
// response like
[{
  id: 'a',
  commandType: ['comd-1', 'cmd-2']
},
{
  id: 'b',
  commandType: ['comd-x', 'cmd-y']
}]
```
