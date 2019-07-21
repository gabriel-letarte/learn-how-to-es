#### number_of_shards: 1

Why does this matter? Because when testing, especially with a very low number of documents, the TF/IDF (calculated per shard) really comes into play for how results are ordered. In order for the ordering to be consistent between tests, it's highly recommended that you set the number of shards to 1.

See [Relevance Is Broken!](https://www.elastic.co/guide/en/elasticsearch/guide/master/relevance-is-broken.html) for more details.
