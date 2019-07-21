# Footnotes

## 1) Edge NGram token filter vs tokenizer
@TODO: Add links!

The Edge NGram token filter is more flexible than the Edge NGram tokenizer. 
1) If you need to remove stop words, it won't be possible to do it after tokenizing in ngrams,
E.g,
`The Quick` would get tokenized into,
```
T
Th
The
Q
```
...

so only `The` would get removed. `T` and `Th` would still be littering your inverted intex.

Moreover, if you have words such as `Italy`, you'll get the following tokens:
```
I
It <-- would get removed by the stop words token filter
Ita
Ital
Italy
```

2) For languages that aren't using spaces as word delimiters, you'll need a language specific tokenizer to split the text into indexable tokens.

3) It's slightly more efficient to do things such as ascii folding and lowercasing on less tokens and then split theses tokens into grams

4) Converting things such `œ` into `oe` needs to produce 2 different grams. If you have `sœur` and ngram it before token filtering

```
s     -> s
sœ    -> soe    ** We're missing `so` as a token
sœu   -> soeu
sœur  -> soeur
```
