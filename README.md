## Tamil-Scientists-search-engine-using-ElasticSearch
This repo contains 100 scientists data scraped from  https://ta.wikipedia.org/wiki. Some of the missing meta data are added manually enhance the dataset.

## Sample JSON data of preprocessed scientist
      {
        "கண்டுபிடிப்பாளர்": "ஐசாக் நியூட்டன்",
        "பிறப்பு": "1643",
        "இறப்பு": "1727",
        "வாழிடம்": "இங்கிலாந்து",
        "தேசியம்": "ஆங்கிலேயர்",
        "துறை": "இயற்பியல்,இயல்,மெய்யியல்,கணிதம்,வானியல்,இரசவாதம்,கிறித்தவ இறையியல்,பொருளியல்",
        "பணியிடங்கள்": "கேம்பிரிச்சுப் பல்கலைக்கழகம்,ரோயல் கழகம்,ரோயல் மின்ட்",
        "கல்வி கற்ற இடங்கள்": "திரித்துவக் கல்லூரி, கேம்பிறிஜ்",
        "அறியப்படுவது": "சர் பட்டம்",
        "கண்டுபிடிப்பு": "நியூட்டனின் ஈர்ப்பு விதி"
      }

Preprocessed JSON data is stored as  the ```Scientists.json``` folder.And the Bulk API format of those 100 scientists are stored as ```bulk_format.txt``` file

The following Query DSL are supported for all the diiferent types of user queries.

##  Query DSL for ElasticSearch search engine

```
 # deleting an index(database)
DELETE /scientists_db
```

This must be run before creating the index(database) because it will not allow you to create index if it already exists.Add the ```analysis``` folder in elasticserach config folder.

#### custom stop words ,stemming and synonyms new analyzer along with the standard analyzer

```
PUT /scientists_db/
{
        "settings": {
            "analysis": {
                "analyzer": {
                    "my_analyzer": {
                        "tokenizer": "standard",
                        "filter": ["custom_stopper","custom_stems","custom_synonym"]
                    }
                },
                "filter": {
                    "custom_stopper": {
                        "type": "stop",
                        "stopwords_path": "analysis/tamil_stopwords.txt"
                    },
                    "custom_stems": {
                        "type": "stemmer_override",
                        "rules_path": "analysis/tamil_stemming.txt"
                    },
                    "custom_synonym": {
                        "type": "synonym",
                        "synonyms_path": "analysis/synonym.txt"                
                    }
                }
            }
        }
    }
```
#### Uploading data using bulk API

```
POST /_bulk
{ "index" : { "_index" : "scientists_db", "_type" : "scientists", "_id" :1 } }
{"கண்டுபிடிப்பாளர்":"பிலைசு பாஸ்கல்","பிறப்பு":"1623","இறப்பு":"1662","வாழிடம்":"பிரான்சு","தேசியம்":"பிரான்சியர்","துறை":"மெய்யியல், கணிதம்","பணியிடங்கள்":"","கல்வி கற்ற இடங்கள்":"பிரான்ஸ் நாட்டு அறிவியல் கலைக்கழகம்","கண்டுபிடிப்பு":"கூட்டும் இயந்திரம்"}
{ "index" : { "_index" : "scientists_db", "_type" : "scientists", "_id" :2 } }
{"கண்டுபிடிப்பாளர்":"ஜான் டால்ட்டன்","பிறப்பு":"1766","இறப்பு":"1844","வாழிடம்":"","தேசியம்":"ஆங்கிலேயர்","துறை":"இயற்பியல்,வேதியல்,வானியல்","பணியிடங்கள்":"","கல்வி கற்ற இடங்கள்":"","அறியப்படுவது":"றோயல் விருது","கண்டுபிடிப்பு":"அணுக் கொள்கை"}
```

#### checking the custom analyzer(stopwords, stemming)
```
GET /scientists_db/_analyze
{
 "text": ["மிகவும் சிறந்த  10  வானியல் துறைசார்ந்த கண்டுபிடிப்பாளர்கள்"],
 "analyzer": "my_analyzer"
}
```

#### checking the similar words
```
GET /scientists_db/_analyze
{
  "text": ["பற்றறி"],
  "analyzer": "my_analyzer"
}
```

#### search for the inventor of "ஹாலோகிராபி"
```
GET /scientists_db/_search
{
 "query": {
   "query_string": {
            "query":"ஹாலோகிராபி"
        }
   }
 }
 ```
 
 #### search for Italian scientists
 ```
 GET /scientists_db/_search
{
 "query": {
   "match": {
           "தேசியம்":"இத்தாலியர்"
        }
   }
}
```

#### search for scientists start with  "ஜா" 
```
GET /scientists_db/_search
{
    "query": {
        "wildcard" : {
	        "கண்டுபிடிப்பாளர்" : "ஜா*"     
	        }
    }
}
```

#### search for scientists end with  "டன்" in their names
```
{
    "query": {
        "wildcard" : {
	        "கண்டுபிடிப்பாளர்" : "*டன்"     
	        }
    }
}
```

#### search for scientists  with  "டன்"  in their names
```
GET /scientists_db/_search
{
    "query": {
        "wildcard" : {
	        "கண்டுபிடிப்பாளர்" : "*கார்*"     
	        }
    }
}
```

#### search for scientists start with ஜேம்ஸ்
```
GET /scientists_db/_search
{
  "query" : {
         "match" : {
              "கண்டுபிடிப்பாளர்":"ஜேம்ஸ்*"
        }
    }
}
```


#### search for scientists studied and worked in cambridge university
```
GET /scientists_db/_search
{
      "query" : {
         "multi_match" : {
             "query" : "கேம்பிரிச் பல்கலைக்கழகம்",
             "fields": ["பணியிடங்கள்","கல்வி கற்ற இடங்கள்"]
         }
     }
}
```

#### search for ஆல்பர்ட் ஐன்ஸ்டைன் spelling mistake - Using custom indexing for search
```
GET /scientists_db/scientists/_search
{
   "query": {
       "multi_match" : {
           "query" : "அல்பர்ட்  ஐன்டன்",
           "fuzziness": "AUTO",
       "analyzer": "my_analyzer"
       }
   }
}
```

#### search for ஆல்பர்ட் ஐன்ஸ்டைன் spelling mistake - Using standard indexing for search
```
GET /scientists_db/scientists/_search
{
   "query": {
       "multi_match" : {
           "query" : "அல்பர்ட்  ஐன்டன்",
           "fuzziness": "AUTO",
       "analyzer": "standard"
       }
   }
}
```

#### search for Italian mathematicians
```
GET /scientists_db/_search
{
"query": {
  "bool": {
        "must": [
            { "match": { "தேசியம்": "இத்தாலியர்" }},
            { "match": { "துறை": "கணிதம்" }}
        ]
      }
    }
 }
 ```
#### search for  physists who received nobel prize
```
GET /scientists_db/_search
{
"query": {
  "bool": {
        "must": [
            { "match": { "அறியப்படுவது":  "நோபல் பரிசு" }},
            { "match": { "துறை": "இயற்பியல்" }}
        ]
      }
    }
}
```
#### search for astronauts died after 2000
```
GET /scientists_db/_search
{
"query": {
    "bool": {
      "must": [{
          "match": {
              "துறை": "வானியல்"
          }
        },
        {
          "range": {
            "இறப்பு" : {
                "gte" : "2000"
            }
          }
        }
      ]
    }
  }
  }
```

#### search for 3 scientists who belongs to the field of வானியல் but not இயற்பியல் studied in oxford university or cambridge university
```
GET /scientists_db/_search
{
 "size":3,
 "query": {
   "bool": {
     "must": {
       "bool" : { 
         "should": [
           { "match": {  "கல்வி கற்ற இடங்கள்": "கேம்பிரிச் பல்கலைக்கழகம்"  }},
           { "match": {  "கல்வி கற்ற இடங்கள்": "ஆக்சுபோர்டு பல்கலைக்கழகம்"  }} 
         ],
         "must": { "match": { "துறை": "வானியல்" }} 
       }
     },
     "must_not": { "match": {"துறை": "இயற்பியல்"  }}
   }
 }
}
```

#### Scientists who received "நோபல்" prize 
```
GET /scientists_db/_search
{
  "query": {
    "multi_match": {
      "query": "நோபில்",
      "fields":["அறியப்படுவது"], 
      "fuzziness": "AUTO",
      "analyzer": "my_analyzer"
    }
  }
}
```

#### bucketing using nationality
```
GET /scientists_db/_search 
{
  "aggs": {
    "group_by_nationality": {
      "terms": { "field": "தேசியம்.keyword" }
    }
  }
}
```

#### bucketing using nationality and prize
```
GET /scientists_db/_search 
{
  "aggs": {
    "group_by_nationality": {
      "terms": {
        "field":  "தேசியம்.keyword"
      }
    },
    "group_by_prize": {
      "terms": {
        "field":  "அறியப்படுவது.keyword"
      }
    }
     
  }
}
```

#### searching for more like scientists names
```
GET scientists_db/_search
{
  "query": {
    "more_like_this" : {
      "fields" : ["கண்டுபிடிப்பாளர்"],
      "like" : "ஜான் நேப்பியர்",
      "min_term_freq" : 1,
      "max_query_terms" : 3
    }
  }
}
```

#### astronaut who is not American but reveived a nobel prize or Bruce Medal but died after 1900
```
GET /scientists_db/_search
{
 "query": {
   "bool": {
     "must": {
       "bool" : { 
         "should": [
           { "match":{"அறியப்படுவது":"நோபல் பரிசு"}},
           { "match":{"அறியப்படுவது":"புரூசு பதக்கம்"} }
         ],
         "filter": [ 
       {
         "range": {
           "இறப்பு" : {
               "gte" : "1900"
           }
         }
       }
     ],
         "must": { "match": { "துறை": "வானியல்" }} 
       }
     },
     "must_not": { "match": {"தேசியம்": "அமெரிக்கர்"}}
   }
 }
}

```
