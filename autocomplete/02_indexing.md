### Indexing the documents

Then we'll index all of our coffee roasts!

```
PUT coffee/_doc/1
{
  "name": "Sumatra"
}

PUT coffee/_doc/2
{
  "name": "Peru"
}

PUT coffee/_doc/3
{
  "name": "French Roast"
}

PUT coffee/_doc/4
{
  "name": "Metta Espresso"
}

PUT coffee/_doc/5
{
  "name": "Peru Small Batch"
}
```

Note, about `PUT coffee/_doc/5`
- `PUT coffee`: Index the given document in the `coffee` index.
- `_doc`: Document types are [deprecated](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/removal-of-types.html), "The preferred type name is `_doc`"
- `/1`: The `_id` if the document
