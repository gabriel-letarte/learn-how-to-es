## Client side solutions

This is a neat solution but it's not perfect. Perhaps you'd like your autocomplete to behave in the following way instead:

> Complete on the last word, Match for previous words.

A quick win is to manually split the tokens on the client side. For example, if we wanted to query for `pe sm` and avoid matching "Peru Small Batch", the following query would yield no results:
```
POST coffee/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "pe"
          }
        },
        {
          "match": {
            "name.edge_ngram": "sm"
          }
        }
      ]
    }
  }
}
```

Changing `pe` for `peru` would yield the expected result. This kind of query would effectively allow completing only on the last term while making sure every word before are found.

This kind of text analysis seems easy to do on the client at first as long as we stay within languages where spaces delimit words. This approach wouldn't work for languages such as Japanese where the usage of spaces is optional.

>>>>> Another solution would be to manually split the input, run the last one against the `edge_ngram` field and the other ones against the regular field. This should work pretty well in languages where the words are delimited by whitespaces but it won't work well in languages such as Japanese or Chinese where tokenizing is a real challenge and can hardly be implemented consistently on both the app and Elasticsearch.
