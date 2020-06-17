# nested query to query nested data types
GET /departments/_search
{
  "query": {
    "nested": {
      "path": "employees",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "employess.position": "intern"
              }
            },
            {
              "term": {
                "employees.gender.keyword": {
                  "value": "F"
                }
              }
            }
          ]
        }
        
      }
    }
    
  }
}

# join_field mappings to define relations betweent 2 docs
PUT /departmensts
{
  "mappings": {
        "properties":{
        "join_field":{
            "type":"join",
            "relations":{
              "department":"employee"
            }
        }
      }
     
  }
}

# join_field mappings
PUT /departmensts
{
  "mappings": {
        "properties":{
        "join_field":{
            "type":"join",
            "relations":{
              "department":"employee"
            }
        }
      }
     
  }
}

# add documents into relations

PUT /departmensts/_doc/1
{
  "name":"Development",
  "join_field":"department"
}

# adding a parent doc
PUT /departmensts/_doc/2
{
  "name":"Marketing",
  "join_field":"department"
}

#adding employees. Routing keys used so that same index shard is used for parent and child docs
PUT /departmensts/_doc/3?routing=1
{
  "name":"Bob",
  "age":29,
  "gender":"m",
  "join_field":{
    "name":"employee",
    "parent":1
  }
}
PUT /departmensts/_doc/6?routing=1
{
  "name":"Nick",
  "age":29,
  "gender":"m",
  "join_field":{
    "name":"employee",
    "parent":1
  }
}
#adding employees
PUT /departmensts/_doc/4?routing=2
{
  "name":"Jon",
  "age":30,
  "gender":"m",
  "join_field":{
    "name":"employee",
    "parent":2
  }
}

#adding employees
PUT /departmensts/_doc/5?routing=2
{
  "name":"Martin",
  "age":30,
  "gender":"m",
  "join_field":{
    "name":"employee",
    "parent":2
  }
}

# has_parent query get all employee of Development type department. i.e. querying child docs by parent
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-has-parent-query.html#_sorting_2
GET /departmensts/_search
{
  "query": {
      "has_parent": {
      "parent_type": "department",
      "query": {
        "term": {
          "name.keyword": {
            "value": "Development"
          }
        }
      }
    }
  }
}


# apply relevance score to child doc
GET /departmensts/_search
{
  "query": {
      "has_parent": {
      "parent_type": "department",
      "score":true,
      "query": {
        "term": {
          "name.keyword": {
            "value": "Development"
          }
        }
      }
    }
  }
}
# has_chilkd query to retrieve parent from child docs
GET /departmensts/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "query": {
        
        "bool": {
          
          "must": [
            {
              "range": {
                "age": {
                  "gte": 20
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": {
                  "value": "M"
                }
              }
            }
          ]
          
        }
        
      }
    }
  }
}

#score mode- min, max, sum, avg modes are used in has_child query for relevance score
GET /departmensts/_search
{
  "query": {
    "has_child": {
      "type": "employee",
	  "score_mode":"sum",
      "query": {
        
        "bool": {
          
          "must": [
            {
              "range": {
                "age": {
                  "gte": 20
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": {
                  "value": "M"
                }
              }
            }
          ]
          
        }
        
      }
    }
  }
}

# apply min_children max_children

GET /departmensts/_search
{
  "query": {
    "has_child": {
      "type": "employee",
	  "score_mode":"sum",
	  "min_children":2,
	  "max_children":4,
	  
      "query": {
        
        "bool": {
          
          "must": [
            {
              "range": {
                "age": {
                  "gte": 20
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": {
                  "value": "M"
                }
              }
            }
          ]
          
        }
        
      }
    }
  }
}

# inner-hits query to same is for has_parent query
GET /departmensts/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "inner_hits": {}, 
      "query": {
        
        "bool": {
          
          "must": [
            {
              "range": {
                "age": {
                  "gte": 20
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": {
                  "value": "M"
                }
              }
            }
          ]
        }
      }
    }
  }
}

#Term lookup mechanism