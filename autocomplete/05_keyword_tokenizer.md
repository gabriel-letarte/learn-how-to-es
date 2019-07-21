## Keyword tokenizer

The [keyword tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-keyword-tokenizer.html) just returns the input as a single token. By using this, we avoid splitting "Peru Small Batch" in 3 tokens, instead, we get a single token: `Peru Small Batch`.
