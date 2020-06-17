#Format result
# query result in yaml format
GET /recipe/_search?format=yaml
{
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}

# query result in in json fromat in readble form
GET /recipe/_search?pretty
{
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}

# Disable _source if not required to reduce result data size
GET /recipe/_search
{
  "_source": false,
  "query": {
    
    "match": {
      "title": "pasta"
    }
    
  }
}
# give field names which needs in _source  
GET /recipe/_search
{
  "_source": "ingredients.name",
  "query": {
    
    "match": {
      "title": "pasta"
    }
    
  }
}
# Use wildcards in _source  
GET /recipe/_search
{
  "_source": "ingredients.*",
  "query": {
    
    "match": {
      "title": "pasta"
    }
    
  }
}

# Use array of fields to be return in _source  
GET /recipe/_search
{
  "_source": ["ingredients.*","servings.max"],
  "query": {
    
    "match": {
      "title": "pasta"
    }
    
  }
}

# Use includes and excludes to filter fields to be in _source  
GET /recipe/_search
{
  "_source":{
    "includes": "ingredients.*",
    "excludes": "ingredients.name"
  },
  "query": {
    
    "match": {
      "title": "pasta"
    }
    
  }
}

# Specifying result (hits) size

# set the return hits to be return to 2 in request uri
GET /recipe/_search?size=2
{
  "_source": false, 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}

# set the return hits to be return to 2 in query dsl
GET /recipe/_search?size=2
{
  "size": 2, 
  "_source": false, 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}

#specifying offset of hits
# set the offset using from parameter.
GET /recipe/_search?size=2
{
  "_source": false, 
  "size": 2, 
  "from": 2, 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}

# pagination
	- use size and from parameter for pagination upto 10000 records.
	-Beyond 10,0000 records you have to use 'search_after' parameter
		link-
	
	formula From = ceil(total_hits/page_size)
	Formula from = (page_size *(page_number-1))

# Sorting
# Sort result
GET /recipe/_search
{
  "query": {
    "match_all": {}
  },
  "_source": false, 
  "sort": [
    {
      "preparation_time_minutes": {
        "order": "desc"
      }
    }
  ]
}

# Sort result using array fields having more than one value. Mode parameter is used for this
GET /recipe/_search
{
  "query": {
    "match_all": {}
  },
  "_source": "ratings", 
  "sort": [
    {
      "ratings": {
        "order": "desc",
        "mode": "avg"
      }
    }
  ]
}

# filter query -query runs in filter and query context. Filter context does not affect relevance score while query context
does. So filter is efficient
GET /recipe/_search
{
  "query": {
    "bool": {
        "must": [
          {
            "match": {
              "title": "pasta"
            }
          }
        ],
        "filter": [
          {
           "range":{
             "preparation_time_minutes":{
               "lte":15
             }
           }
          }
          ]
    }
  }
}