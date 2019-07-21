#### fields

Using [fields aka multi_fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html) have a few nice properties. 
- it guarantees the nested fields are built from the same source value
- the ES response isn't littered with different versions of the field containing the same _source value.
- @TODO: Not actually sure about this. Need to investigate more. it stores less data on disk (only store one `_source` field)
