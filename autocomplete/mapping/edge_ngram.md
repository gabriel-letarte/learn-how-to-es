#### name

This field would be used for non autocomplete related use cases

#### name.edge_ngram

This field will be queried with partial words.

#### analyzer.edge_ngram

This analyzer will ngram the words starting at the edge to dispose text in a way that makes it efficient to perform prefix queries.

Example with the word `coffee`:

```
c
co
cof
coff
coffe
coffee
```

_See this for more details about [ngram filter vs tokenizer](/autocomplete/mapping/edge_ngram_filter_vs_tokenizer.md)._

#### search.search

This analyzer will be applied to the search queries. We wouldn't want to apply ngram on the terms, else we'll end up matching a whole lot of junk. E.g, `cold` would yield:

co
col
cold

and would suggest coffee because of the first gram (`co`).

_See this for more details about [the need for a search_analyzer](/autocomplete/mapping/search_analyzer.md)._

#### min 1

This will be the minimum number of characters we'll need to type before we can start returning results.

Starting with 1 is usefull for a small number of potential results but quickly becomes irrelevant for very large quantities of results. I still suggest going with 1 for a better and more consistent user experience no matter how many number of potential results because seeing results reinforces that _there is_ results starting with that first character.

#### max 20

This is the max number of characters we'll be able to look at to give a suggestion. When setting this number, you have to ask yourself, does it really make sense to continue giving suggestions after having typed 20 consecutive characters?
