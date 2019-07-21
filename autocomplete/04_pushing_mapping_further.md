### Pushing Mapping Furhter

An index needs to be closed before changing it's settings:
```
POST coffee/_close
```

Adding our new analyzer:
```
PUT coffee/_settings
{
  "analysis": {
    "analyzer": {
      
      "edge_ngram_single_token": {
        "tokenizer": "keyword",
        "filter": [
          "lowercase",
          "edge_ngram_filter"
        ]
      },
      
      "search_single_token": {
        "tokenizer": "keyword",
        "filter": [
          "lowercase"
        ]
      }
      
    }
  }
}
```

Adding our field:
```
PUT coffee/_mapping/_doc
{
  "properties": {
    "name": {
      "type": "text",
      "fields": {

        "edge_ngram_single_token": {
          "type": "text",
          "analyzer": "edge_ngram_single_token",
          "search_analyzer": "search_single_token"
        },

      }
    }
  }
}
```

Reopening the index to be able to query it:
```
POST coffee/_open
```

Update the existing documents from `_source` (see [`_update_by_query`](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html) for more details):
```
POST coffee/_update_by_query
```

A quick test to verify our mapping has been properly updated
```
GET coffee
```

A quick test to see how our new analyzers behaves:
```
POST coffee/_analyze
{
  "text": "Peru Small Batch",
  "analyzer": "edge_ngram_single_token"
}
```

```
pe
per
peru
peru_
peru s
peru sm
peru sma
...
```

```
POST coffee/_analyze
{
  "text": "Peru Sm",
  "analyzer": "search_single_token"
}
```

```
peru sm
```

Time for a quick test:
```
POST coffee/_search
{
  "query": {
    "match": {
      "name.edge_ngram_single_token": {
        "query": "peru sm",
        "operator" : "and"
      }
    }
  }
}
```
```json
{
  "hits": [
    {
      "_source": {
        "name": "Peru Small Batch"
      }
    }
  ]
}
```

Works exactly as we expected. Now, in order to find a coffee by name, you'll have to start at the beginning of the title. Which means searching for `small` wouldn't find the coffee.

We have a few things to talk about now. The usage of the `keyword` tokenizer, the need to have a `search_analyzer` which we didn't talk about earlier, some potential techniques to push this concept further and things we could do client side to avoid using this technique.
