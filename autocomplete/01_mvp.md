## What's an autocomplete?

When I ask people this question, I often get

> Well it completes your words in the search box

At this point, I need to give a bit more context. Let's say we're working for a fine coffee roaster that sells 50 different blends.

Ok yes, where does it get the words from?

> The list of coffee of course!

Right... let's get to work!




## The MVP

We will keep the example really simple for the MVP. Let's assume we only represent each roast in the database by it's name and id. Our coffee documents looks like this:

```json
{
  "id": 1,
  "name": "Sumatra"
}
```

Looks good, let's dive into actual Elasticsearch code!




### The mapping

First thing you need to do is define your mapping so you'll be able to insert your data in Elasticsearch.

`id`: Let's use the elasticsearch [`_id` field](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-id-field.html) to store our ids automatically.

`name`: We'll go right ahead and use the [Edge NGram token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenfilter.html) to analyze our coffee names in an efficient way for completion.

The mapping might look like this:

```
PUT coffee
{
  "settings": {
    "number_of_shards": 1,
    "analysis": {

      "filter": {
        "edge_ngram_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20
        }
      },

      "analyzer": {
        "edge_ngram": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding",
            "edge_ngram_filter"
          ]
        },
        "search": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
      
    }
  },
  "mappings": {
    "_doc": {
      "properties": {
        
        "name": {
          "type": "text",
          "fields": {
            "edge_ngram": {
              "type": "text",
              "analyzer": "edge_ngram",
              "search_analyzer": "search"
            }
          }
        }

      }
    }
  }
}
```

A few things about this:
- About using [fields](/autocomplete/mapping/fields.md)
- Details about our [edge_ngram settings](/autocomplete/mapping/edge_ngram.md)
- Always use a [single master shard](/autocomplete/mapping/number_of_shards.md) in dev
