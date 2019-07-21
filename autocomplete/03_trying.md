### Trying it

```
POST coffee/_search
{
  "query": {
    "match": {
      "name.edge_ngram": {
        "query": "per",
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
      "_id": "2",
      "_score": 1.3635619,
      "_source": {
        "name": "Peru"
      }
    },
    {
      "_id": "5",
      "_score": 1.1762023,
      "_source": {
        "name": "Peru Small Batch"
      }
    }
  ]
}
```

It works as expected. `Peru` was analyzed and saved as:

```
pe
per <- This one was our hit
peru
```

Note that the "Peru" roast has scored higher than the "Peru Small Batch" roast because `Peru` is the only term within the title of "Peru" and there is 3 terms in "Peru Small Batch".

Now we can expose the feature via an endpoint. Whenever we get a hit, we'll return the name of the coffee and call it a day right? Well, that wouldn't be the best experience. Let's take a look why.

One more thing we should test.

```
POST coffee/_search
{
  "query": {
    "match": {
      "name.edge_ngram": {
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

Still looking good. Now let's try another thing:

```
POST coffee/_search
{
  "query": {
    "match": {
      "name.edge_ngram": {
        "query": "sm ba",
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

That one is more questionnable but makes a lot of sense technically, here's why. We've sent 2 tokens to Elasticsearch, `sm` and `ba`.

```
pe
per
peru

sm <- hit
sma
small

ba <- hit
bat
batc
batch
```

But from a UX perspective it's confusing.

A potential solution would be to avoid tokenizing the input at index time before we Edge NGram it. This would have the effect of always completing from the beginning. Let's take a look.
