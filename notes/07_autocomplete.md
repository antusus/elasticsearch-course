# N-grams

When term is analyzed in a field it is split into n-grams for example for word `star`:

- unigram: [s, t, a, r]
- bigram: [st, ta, ar]
- trigram: [sta, tar]
- 4-gram: [star]

when trying to implement search by prefix by typing `tar` word `star` will be returned since there is a trigram created for it. There is
thing called `edge n-gram` which are calculated only from the beginning of the word.

# Autocomplete analyzer with Edge N-Grams

If we want to have edge n-grams calculated we have to specify this during index creation.

```
PUT /movies

{
    "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20
        }
    },
    "analyzer": {
        "autocomplete": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "autocomplete_filter"
          ]
        }
    }
    "mappings": {
        "properties": {
          "title": {
            "type": "text",
            "analyzer": "autocomplete"
          }
        }
    }         
}
```

Still with query `star tr` results will contain "Star Trek" and "Star Wars" since part
of the query `star` matches. This is where we can use completion suggester.

# search_as_you_type field

Type optimized to provide support for search as you type completion use case. It creates a series of subfields that are analyzed to index
terms that can be efficiently matched by a query that partially matches the entire indexed text value. It supports both: `prefix`
and `infix` matching.

To define it:

```
PUT /movies
{
  "mappings": {
    "properties": {
      "title": {
        "type": "search_as_you_type",
        "analyzer": "english",
      }
    }
  }
}
```

This creates following fields:
- **title** - analyzed and configured as in the mapping (in our example we are using "english" analyzer),
- **title._2_gram** - wraps the field with a shingle token of shingle size 2,
- **title._3_gram** - wraps the field with a shingle token of shingle size 3,
- **title._index_prefix** - wraps the analyzer of `*title._3_gram` with an edge ngram token filter.

You can specify size of shingled with `max_shingle_size`, default value is 3. Valid values are `2,3,4`.

