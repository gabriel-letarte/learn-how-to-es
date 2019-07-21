#### name

This field would be used for non autocomplete related use cases

#### name.edge_ngram

This field will be queried with partial words.

#### analyzer.edge_ngram

This analyzer will ngram the words starting at the edge to dispose text in a way that makes it efficient to perform prefix queries. See _(see Footnote #1)_ for more details about ngram filter vs tokenizer.  Example with the word `coffee`:

```
co
cof
coff
coffe
coffee
```

#### search.search

This analyzer will be applied to the search queries. We wouldn't want to apply ngram on the terms, else we'll end up matching a whole lot of junk. E.g, `cold` would yield:

co
col
cold

and would suggest coffee because of the first gram (`co`).

#### min 2

This will be the minimum number of characters we'll need to type before we can start returning results. Choosing 1 is usefull for a small number of potential results but quickly becomes irrelevant for very large quantities of results.

#### max 20

This is the max number of characters we'll be able to look at to give a suggestion. When setting this number, you have to ask yourself, does it really make sense to continue giving suggestions after having typed 20 consecutive characters?
