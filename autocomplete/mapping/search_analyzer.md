## The need for a `search_analyzer`

First, let's create 2 other fields to illustrate why we need a search analyzer. We'll also get to play with the infamous Edge NGram Tokenizer.

We need to go through the same index update dance:
```
POST coffee/_close

PUT coffee/_settings
{
  "analysis": {
    "tokenizer": {
      
      "my_edge_ngram": {
        "type": "edge_ngram",
        "min_gram": 1,
        "max_gram": 20
      }

    },
    
    "analyzer": {
      
      "edge_ngram_tokenizer": {
        "tokenizer": "my_edge_ngram",
        "filter": [
          "lowercase"
        ]
      }
      
    }
  }
}

PUT coffee/_mapping/_doc
{
  "properties": {
    "name": {
      "type": "text",
      "fields": {

        "bad_config": {
          "type": "text",
          "analyzer": "edge_ngram"
        },

        "bad_config_single_token": {
          "type": "text",
          "analyzer": "edge_ngram_single_token"
        },
        
        "bad_config_edge_ngram_tokenizer": {
          "type": "text",
          "analyzer": "edge_ngram_tokenizer"
        }

      }
    }
  }
}

POST coffee/_open

POST coffee/_update_by_query
```

And now let's see why _always_ we need a `search_analyzer` when we analyze a field using a filter that could yield multiple terms.

```
POST coffee/_search
{
  "query": {
    "match": {
      "name.bad_config_single_token": {
        "query": "peru sm",
        "operator": "and",
        "minimum_should_match": "100%"
      }
    }
  }
}
```
```json
{
  "hits": [
    {
      "_score": 1.8074193,
      "_source": {
        "name": "Peru Small Batch"
      }
    },
    {
      "_score": 1.7025691,
      "_source": {
        "name": "Peru"
      }
    }
  ]
}
```

What the ..., you might wonder? I'm supposed to have a single `keyword` no? If you want to add the confusion, the exact same query on the `name.edge_ngram` will only yield `Peru Small Batch`. That's pretty much the opposite of what I would expect before digging in.

Although it's not properly documented, here's how Elasticsearch would perform both queries.

### Query on `name.bad_config_single_token`

The given term will be transformed in the following tokens:
```
p
pe
per
peru
peru_
peru s
peru sm
```

At which point the query will be rewritten internally and, _appart the fact it's still going to be scored_, it will behave like a [`terms` query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-terms-query.html). Terms query will find every document matching _at least_ one of their terms. Here's a way of thinking about this:

```
POST coffee/_search
{
  "query": {
    "terms": {
      "name.bad_config": ["p", "pe", "per", "peru", "peru ", "peru s", "peru sm"]
    }
  }
}
```

Inverted index:
```
p        -> doc4, doc5  <= `p` hits doc 4 & 5
pe       -> doc4, doc5  <= `pe` hits doc 4 & 5
per      -> doc4, doc5  <= `per` hits doc 4 & 5
peru     -> doc4, doc5  <= `peru` hits doc 4 & 5
peru_    -> doc5        <= `peru ` hits doc 5
peru s   -> doc5        <= `peru s` hits doc 5
peru sm  -> doc5        <= `peru sm` hits doc 5
```

### Query on `name.edge_ngram`

Now, why do `name.edge_ngram` doesn't have this problem? Well it has it has a problem but it's slightly different. The query would be rewritten like this:

```
POST coffee/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "terms": {
            "name.edge_ngram": ["p", "pe", "per", "peru"]
          }
        },
        {
          "terms": {
            "name.edge_ngram": ["sm"]
          }
        }
      ]
    }
  }
}
```

Inverted index:
```
p        -> doc4, doc5  <= `p` hits doc 4 & 5
pe       -> doc4, doc5  <= `pe` hits doc 4 & 5
per      -> doc4, doc5  <= `per` hits doc 4 & 5
peru     -> doc4, doc5  <= `peru` hits doc 4 & 5

sm  -> doc5        <= `sm` hits doc 5
```

First clause returns `doc4, doc5` and second clause returns `doc5`. The intersection of both clauses returns `doc5`.

This becomes a problem when you query, for example: `permit`. Where you would match both "Peru" and "Peru Small Batch" because:
```
p        -> doc4, doc5  <= `p` hits doc 4 & 5
pe       -> doc4, doc5  <= `pe` hits doc 4 & 5
per      -> doc4, doc5  <= `per` hits doc 4 & 5
```
`perm`, `permi`, `permit` hits nothing but `terms` will yield every document matching _at least one_ term. And in this case, both match two of the analyzed terms.

### Driving the point home

Let's make one more test.

```
POST coffee/_search
{
  "query": {
    "match": {
      "name.bad_config_edge_ngram_tokenizer": {
        "query": "peru sm",
        "operator": "and",
        "minimum_should_match": "100%"
      }
    }
  }
}
```
```json
{
  "hits": [
    {
      "_index": "coffee",
      "_source": {
        "name": "Peru Small Batch"
      }
    }
  ]
}
```

As expected, the tokenizer yielded a single term because the query is rewritten to something like:

```
POST coffee/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "terms": {
            "name.edge_ngram": ["p"]
          }
        },
        {
          "terms": {
            "name.edge_ngram": ["pe"]
          }
        },
        ...
      ]
    }
  }
}
```

Inverted index:
```
p        -> doc4, doc5  <= `p` hits doc 4 & 5
pe       -> doc4, doc5  <= `pe` hits doc 4 & 5
per      -> doc4, doc5  <= `per` hits doc 4 & 5
peru     -> doc4, doc5  <= `peru` hits doc 4 & 5
peru_    -> doc5        <= `peru ` hits doc 5
peru s   -> doc5        <= `peru s` hits doc 5
peru sm  -> doc5        <= `peru sm` hits doc 5
```

Intersect
```
p: (4, 5)
pe: (4, 5)
per: (4, 5)
peru: (4, 5)
peru_: (5)
peru s: (5)
peru sm: (5)
```

Result: Doc 5


### Conclusion

Tokens yielded by the tokenizer each get's their own clauses and tokens yielded by the token filters are queries for in a `terms` query like fashion.

E.g,

`peru sm`
Tokenized into
`peru` AND `sm`
Token filters
(`p` or `pe` or `per` or `peru`) AND (`s`, `sm`)
