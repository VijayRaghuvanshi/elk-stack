### ES cluster related queries

#_cluster api
GET /_cluster/health

#_cat api
GET /_cat/nodes?v
GET /_cat/indices?v

###Create Index###
--create simple index 'pages' with deafault settings replica shard -1, replication -1,
PUT /pages

# create index products with shard and replica config.
PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}

#add a doc in products index
POST /products/_doc
{
  "name":"coffee maker",
  "price": 64,
  "in_stock":10  
}

## response
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "nwEKa3IBYDpek9wCS9pb",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 3,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}



# retrieve the documnet
GET /products/_doc/100

#response
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Toster",
    "price" : 49,
    "in_stock" : 3
  }
}

# scripted update with --
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}
# scripted update assign value
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock = 10"
  }
}

# scripted update value using params
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}

# scripted update value using if stmt
POST /products/_update/100
{
  "script": {
    "source": """
    if(ctx._source.in_stock >= 7)
    {
      ctx._source.in_stock--;
    }else{
      
      ctx._source.in_stock++;
    }
    """
  }
}


# upsert update --update if doc exist with given id otherwise create new one with upsert
POST /products/_update/101
{
  "script": {
        "source": "ctx._source.in-stock--"
  }
  , "upsert": {
      
      "name":"Blender",
      "price":399,
      "in_stock":5
    }
}

GET /products/_doc/101


# add new fields in existing documnet with id 100
POST /products/_update/100
{
  "doc": {
    "tag":["electronics"]
    }
}


# update the documnet with id 100
POST /products/_update/100
{
  "doc": {
    "in_stock":3
    }
}

# retrieve the documnet
GET /products/_doc/100

#add a doc in products with id 100. Change Post to  put
PUT /products/_doc/100
{
  "name":"Toster",
  "price": 49,
  "in_stock":4  
}

#add a doc in products index
POST /products/_doc
{
  "name":"coffee maker",
  "price": 64,
  "in_stock":10  
}



PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}

#DELETE an index
DELETE /pages
PUT /pages

##_cluster api
GET /_cluster/health

###_cat api
GET /_cat/nodes?v
GET /_cat/indices?v
GET /_cat/shards?v

#

# Replace document existing price 49 changed to 79
PUT /products/_doc/100    
{
  "name":"Toaster",
  "price":79,
  "in_stock":4
}

# Deleting a doc WITH ID 101
DELETE /products/_doc/101


#update multiple docs in single query
# "conflicts":"proceed" - allow to proceed in case of version conflict
POST /products/_update_by_query
{
  "conflicts":"proceed", 
  "script":{
    "source":"ctx._source.in_stock--"
  },
  "query":{
    "match_all":{}
  }
}

#Delete multiple docs in single query. Here delete all docs in products index
POST /products/_delete_by_query
{
  "query":{
    "match_all":{}
  }
}


# Bulk API to perform index, create, update, delete action in single query
POST /_bulk
{"index":{"_index":"products","_id":200}}
{"name":"Expresso Machine", "price":199,"in_stock":5}
{"create":{"_index":"products", "_id":201}}
{"name":"Milk Powder", "price":149,"in_stock":14}


POST /_bulk
{"update":{"_index":"products", "_id":201}}
{"doc":{"price":129}}
{"delete":{"_index":"products", "_id":200}}


POST /products/_bulk
{"update":{"_id":201}}
{"doc":{"price":129}}
{"delete":{"_id":200}}


###Analyze api using standard analyzer###
POST /_analyze
{
  "text": ["2guys walk into a bar, but the third...DUCK!...-:)"]
  , "analyzer": "standard"
}
###Analyze api using character filter, tokenizer, toker filter instead of an analyzer###
POST /_analyze
{
  "text": ["2guys walk into a bar, but the third...DUCK!...-:)"]
  , "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}

###Create an index reviews with mappings.
PUT /reviews
{
  "mappings": {
    
    "properties": {
      "rating":{"type": "float"},
       "content":{"type": "text"},
       "product_id":{"type": "integer"},
       "author_id":{
         "properties": {
           "first_name":{"type":"text"},
           "last_name":{"type":"text"},
           "email":{"type":"keyword"}
          }
      }
    }
  }
}

###Create an index reviews with mappings dot notation.
PUT /reviews_dot_notation
{
  "mappings": {
    
    "properties": {
      "rating":{"type": "float"},
       "content":{"type": "text"},
       "product_id":{"type": "integer"},
       "author_id.first_name":{"type":"text"},
        "author_id.last_name":{"type":"text"},
        "author_id.email":{"type":"keyword"}
          
      }
    }
  }
   
###Retrieve mapping
GET /reviews/_mapping

###Retrieve mapping of a field
GET /reviews/_mapping/field/content

###Retrieve mapping of a field with dot notation
GET /reviews/_mapping/field/author_id.email



### add new field in existing index  
PUT /reviews/_mapping
{
  "properties":{
    "created_at":{
      "type":"date"
    }
  }
}

#### ADD DATE FIELDS ###
PUT /reviews/_doc/2
{
  "rating":4.5,
  "content":"Not bad",
  "product_id":123,
  "created_at":"2015-03-27",
  "author":{
    "first_name":"Chris",
    "last_name":"Martin",
    "email":"chris@gmail.com"
  }
}
PUT /reviews/_doc/2
{
  "rating":4.5,
  "content":"Not bad",
  "product_id":123,
  "created_at":"2015-03-27T01:47:23Z",
  "author":{
    "first_name":"Chris",
    "last_name":"Martin",
    "email":"chris@gmail.com"
  }
}
  
 PUT /reviews/_doc/3
{
  "rating":4.5,
  "content":"Not bad",
  "product_id":123,
  "created_at":"2015-03-27T01:47:23+01:00",
  "author":{
    "first_name":"Chris",
    "last_name":"Martin",
    "email":"chris@gmail.com"
  }
}
  

### Re-index index of all doc or source index into destination index also change _source data type to to string for product_id
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"  
  },
  "script": {
    "source": """
      if(ctx._source.product_id != null)
      {
        ctx._source.product_id=ctx._source.product_id.toString();
      }
    """
  }
}

# Using Field alias
PUT /reviews/_mapping
{
  "properties":{
    
    "comment":{
      "type":"alias",
      "path":"content"
    }
  }
}

## mapp a filed to multiple mapping like ingredients here  map to text and keyword type
PUT /multi_field_test
{
  "mappings": {
    
    "properties": {
      
      "desc":{
        "type": "text"
      },
      "ingredients":{
        "type": "text",
        "fields":{
          "keyword":{
            "type":"keyword"
          }
        }
      }
    }
  }
}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   

# create a template access-log, could be used where we have to create tomcat access log indexes using defined mappings and settings.
PUT /_template/access-log
{
  "index_patterns": ["access-log-*"], 
  "settings": {
    "number_of_replicas": 1,
    "index.mapping.coerce":false
    
  }, 
  "mappings": {
    "properties": { 
    "@timestamp": { "type":"date"},
    "url.original":{"type":"keyword"},
    "http.request.referer":{"type":"keyword"},
    "http.response.status_code":{"type":"long"}
    }
  }
  
}

# create a template access-log, could be used where we have to create tomcat access log indexes using defined mappings and settings.
PUT /_template/access-log
{
  "index_patterns": ["access-log-*"], 
  "settings": {
    "number_of_replicas": 1,
    "index.mapping.coerce":false
    
  }, 
  "mappings": {
    "properties": { 
    "@timestamp": { "type":"date"},
    "url.original":{"type":"keyword"},
    "http.request.referer":{"type":"keyword"},
    "http.response.status_code":{"type":"long"}
    }
  }
  
}

#create an index using that will use upper defined template automatically
PUT /access-log-2020-06-05

#Get Index details an check
GET /access-log-2020-06-05

#Delete an index template
DELETE /_template/access-log

#Elastic Common Schema(ECS)
The Elastic Common Schema (ECS) is an open source specification, developed with 
support from the Elastic user community. ECS defines a common set of fields to be
 used when storing event data in Elasticsearch, such as logs and metrics.
 ECS also provides a set of naming guidelines for adding custom fields.
 
https://www.elastic.co/guide/en/ecs/current/ecs-reference.html
https://www.elastic.co/guide/en/ecs/current/ecs-guidelines.html

#Stemming and stop words

1. Stemming is the process to change a word to its original form
	for example "drinking" -> "drink"
				"loved" ->	"love"
2. Stop words stop some of irrelevant words from being indexed
	for eg. a, an, the, 's, etc.
3. ES use analyzer for this.
`
#ES built in analyzer
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html

# Creating custom analyzer with name my_custom_analyzer

PUT /analyzer_test
{
  "settings":{
    "analysis":{
      "analyzer":{
        "my_custom_analyzer":{
          "type":"custom",
          "char_filter":["html_strip"],
          "tokenizer":"standard",
          "filter":[
            "lowercase",
            "stop",
            "asciifolding"
            ]
        }
      }
    }
  }
}

#analyzer custom analyzer
POST /analyzer_test/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text":"I&apos;m in a <em>good</em> mood."
}

# close api 
 -Close api is used to stop index serving search and index requests.
 
 POST /<index_name>/_close
 
# open api 
 -Open api is used to open a closed index serving search and index requests.
  POST /<index_name>/_open 
  
  
#Request URI searching
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html

#Searching using DSL


# Term query variation 1 
GET /products/_search
{
  "query": {
    "term": {
      "in_stock":5
    }
  }
}

# Term query variation 2
GET /products/_search
{
  "query": {
    "term": {
      "in_stock":{
        "value": "5"
      }
    }
  }
}  
# Term query with multiple terms
GET /products/_search
{
  "query": {
    
    "terms": {
      "tags.keyword": [
        "Soup",
        "Cake"
      ]
    }
  }
}

#Fetch documents based on ids.
GET /products/_search
{
  "query": {
    "ids": {
      "values": ["1","2","3"]
    }
    
  }
}

GET /products/_search
{
  "query": {
    "term": {
      "in_stock":5
    }
  }
}

GET /products/_search
{
  "query": {
    "term": {
      "in_stock":{
        "value": "5"
      }
    }
  }
}


GET /products/_search
{
  "query": {
    
    "terms": {
      "tags.keyword": [
        "Soup",
        "Cake"
      ]
    }
  }
}

GET /products/_search
{
  "query": {
    
    "ids": {
      
      "values": ["1","2","3"]
    }
    
  }
  
}


#Range Query
GET /products/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte": 10,
        "lte": 20
      }
    }
    
  }
}

#Range Query with date
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01",
        "lte": "2010/12/31"
      }
    }
    
  }
}
GET /products/_search
{
  "query": {
    "term": {
      "in_stock":5
    }
  }
}

GET /products/_search
{
  "query": {
    "term": {
      "in_stock":{
        "value": "5"
      }
    }
  }
}


GET /products/_search
{
  "query": {
    
    "terms": {
      "tags.keyword": [
        "Soup",
        "Cake"
      ]
    }
  }
}

GET /products/_search
{
  "query": {
    
    "ids": {
      
      "values": ["1","2","3"]
    }
    
  }
  
}


#Range Query
GET /products/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte": 10,
        "lte": 20
      }
    }
    
  }
}

#Range Query with date
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01",
        "lte": "2010/12/31"
      }
    }
    
  }
}
#Range Query with date custom format
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "01/01/2010",
        "lte": "31/12/2010"
        , "format": "dd/MM/yyyy"
      }
    }
    
  }
}

# Relative dates. We can use + and - year, month and day etc. from anchor date (2010/01/01)
GET /products/_search
{
  "query": {
    
    "range": {
      "created": {
        "gte": "2010/01/01||-y-d"
      }
    }
    
  }
}

# Exists query used to get non-null values
GET /products/_search
{
  "query": {
   "exists": {
    "field": "tags"
    }
  }
}

# prefix term query
GET /products/_search
{
  "query": {
    "prefix": {
      "tags.keyword": {
        "value": "Vege"
      }
    }
  }
}

# wildcard query - ?, *
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": {
        "value": "Vege*ble"
      }
    }
  }
}
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": {
        "value": "Vege?ble"
      }
    }
  }
}


# Regular Expression query
GET /products/_search
{
  "query": {
    "regexp": {
      "tags.keyword": "Veg[a-zA-Z]+ble"
    }
  }
}





























