# aggregation query -single value aggregation such as min, max, sum, avg.
# setting size =0 does not result query result only aggregation result

GET /order/_search
{
  "size": 0,
  "aggs": {
    "total_sales": {
      "sum": {
        "field":"total_amount"
      }
    },
    "avg_sale":{
      "avg":{
        "field":"total_amount"
      }
    },
    "min_sale":{
      "min":{
        "field":"total_amount"
      }
    },
    "max_sale":{
      "max":{
        "field":"total_amount"
      }
    }
  }
}

# cardinality aggregation- tells approximate unique value of given field
GET /order/_search
{
  "size": 0,
  "aggs": {
    "total_salesman":{
    "cardinality": {
      "field": "salesman.id"
    }
    }
 }
}

# value_count aggregation- count of diff values used in agg. i.e count of total_amount fields in order index
GET /order/_search
{
  "size": 0,
  "aggs": {
    "count_total_amt":{
    "value_count": {
      "field": "total_amount"
    }
    }
 }
}

# Stats - multi value agg uses min, max, sum, avg in single agg i.e. stats

GET /order/_search
{
  "size": 0,
  "aggs": {
    "total_amt_stats": {
      "stats": {
        "field":"total_amount"
      }
    }
  }
  
}

# bucket agg- term aggregation on filed status
GET /order/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field":"status.keyword",
        "missing":"N/A"
      }
    }
  }
}

# bucket agg- term - define min number of doc count in a bucket
GET /order/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field":"status.keyword",
        "missing":"N/A",
        "min_doc_count":200
      }
    }
  }
}

# bucket agg- term - define order on bucket being created - use order _key
GET /order/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field":"status.keyword",
        "missing":"N/A",
        "min_doc_count":0,
        "order":{
          "_key":"asc"
        }
        
      }
    }
  }
}

# Nested agg - stat agg inside term agg, by default agg usses match all query. agg runs in query context which is match_all by default otherwise you can define your own quer.
GET /order/_search
{
  "query": {
    "match_all": {}
  }, 
  "size": 0,
  "aggs": {
    "term_agg": {
      "terms": {
        "field":"status.keyword"
      },
      "aggs":{
        "stats_status":{
          "stats":{
            "field":"total_amount"
          }
          
        }
      }
    }
  }
}

# Nested agg - filter out documents using filter in agg

GET /order/_search
{
  "query": {
    "match_all": {}
  }, 
  "size": 0,
  "aggs": {
    
    "low_value":{
      "filter":{
        "range":{
          "total_amount":{
            "lt":50
          }
        }
      },
      "aggs":{
        "avg_amount":{
      "avg":{
       
          "field":"total_amount"
        
      }
        }
      }
      
    }
    
    
  }
}


# Range aggs
GET /order/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "range":{
        "field":"total_amount",
        "ranges":[
          {
            "to":50
          },
           {
            "from":50,
            "to":100
          },
          {
            "from":100
          }
        ]
        
      }
  }
  }
}
# Date_Range aggs
GET /order/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range":{
        "field":"purchated_at",
        "ranges":[
          {
            "from":"2016-01-01",
            "to":"2016-01-01||+6M"
          },
          {
            "from":"2016-01-01||+6M",
            "to":"2016-01-01||+1y"
          }
        ]
        
      }
  }
  }
}

# Date_Range aggs with date format
GET /order/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range":{
        "field":"purchated_at",
        "format":"yyyy-MM-dd",
        "ranges":[
          {
            "from":"2016-01-01",
            "to":"2016-01-01||+6M"
          },
          {
            "from":"2016-01-01||+6M",
            "to":"2016-01-01||+1y"
          }
        ]
        
      }
  }
  }
}

# Date_Range aggs with date format and set bucket name with keyed=true
GET /order/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range":{
        "field":"purchated_at",
        "format":"yyyy-MM-dd",
        "keyed":true,
        "ranges":[
          {
            "from":"2016-01-01",
            "to":"2016-01-01||+6M"
          },
          {
            "from":"2016-01-01||+6M",
            "to":"2016-01-01||+1y"
          }
        ]
        
      }
  }
  }
}

# use nested agg and add stats aggs for each bucket
GET /order/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range":{
        "field":"purchated_at",
        "format":"yyyy-MM-dd",
        "keyed":true,
        "ranges":[
          {
            "from":"2016-01-01",
            "to":"2016-01-01||+6M"
          },
          {
            "from":"2016-01-01||+6M",
            "to":"2016-01-01||+1y"
          }
        ]
      },
      "aggs":{
        "bucket_stats":{
          "stats":{
            "field":"total_amount"
          }
        }
      }
  }
  }
}

# histogram agg - define intervals for buckets dynamically
GET /order/_search
{
  "size":0,
  "aggs": {
    "amount_distribution": {
      "histogram": {
        "field":"total_amount",
        "interval":25
      }
    }
  }
  
}


# histogram agg - define histogram boundary
GET /order/_search
{
  "size":0,
  "aggs": {
    "amount_distribution": {
      "histogram": {
        "field":"total_amount",
        "interval":25,
        "min_doc_count":0,
        "extended_bounds":{
          "min":0,
          "max":500
        }
      }
    }
  }
  
}

# date_histogram agg - for dates fields
GET /order/_search
{
  "size":0,
  "aggs": {
    "orders_over_time": {
      "date_histogram": {
        "field":"purchased_at",
        "format":"yyyy-MM-dd",
        "interval":"month",
        "min_doc_count":0,
        "extended_bounds":{
          "min":"2016-01-01",
          "max":"2017-12-31"
        }
      }
    }
  }
}

# global aggs - it does not influence by query context. It uses match_all query always irrespective what have defined in query clause. Use global
# global aggs can be placed at top level only.
GET /order/_search
{
  "query": {
    
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "size": 0,
  "aggs": {
    "stats_expensive": {
      "stats": {
        "field":"total_amount"
      }
    },
    
  "all_orders":{
    "global":{},
    "aggs":{
      "stats_amt":{
        "stats":{
        "field":"total_amount"
      }
      }
    }
  }
  }
}
# Missing aggs - used for the those scenarios where a field value is null or does not exist
# For e.g. let say status field not defined for all the docs
GET /order/_search
{
  "size": 0,
  "aggs": {
    "missing_status": {
      "missing": {
        "field":"status.keyword"
      }
    }
  }
}

# Nested agg -for nested objects
GET /departmensts/_search
{
  "size": 0,
  "aggs": {
    "employees": {
      "nested": {
        "path":"employess"
      },
     
    "aggs": {
      "minimum_age": {
        "min": {
          "field":"employee.age"
        }
      }
    }
  }
  }
}







