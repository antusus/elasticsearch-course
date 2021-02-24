# Search

## Queries and Filters

Query will return data based on relevance. Filters are checking if data exists. They are also cacheable. Example:

```json
{
  "query": {
    "bool": {
      "must": {
        "term": {
          "title": "trek"
        }
      },
      "filter": {
        "range": {
          "year": {
            "gte": 2010
          }
        }
      }
    }
  }
}
```

We are using boolean query to create a query, and we are using filter to limit amount of data we want to check.

### Filters

Filters are executed in a separate _filter context_ where scoring is ignored, but they can be considered for caching. Sample filters:

- **term**: filter by exact value `{"term": {"year": 2010}}`
- **terms**: filter if any values in a list match `{"terms": {"year": [2010, 2011]}}`
- **range**: find numbers or dates in a given range (gt, lt, gte, lte) `{"range": {"year": {"gte": 2010}}}`
- **exists**: find documents with a field `{"exits": {"field": "tags"}}`
- **missing**: find documents with a missing field `{"missing": {"field": "tags"}}`
- **bool**: combine filters with boolean logic (must, must_not, should)

### Queries

Queries are calculating score for each element and returns them in order of relevance. Some queries:

- **match_all** - returns all documents `{"match_all": {}}`
- **match** - searches analyzed results `{"match": {"title": "star"}}`
- **multi_match** - run the same query on different fields `{"multi_match": {"query": "star", "fields": ["title", "synopsis"]}}`
- **bool** - queries documents matching boolean combinations of other queries. Takes
  _more-matches-is-better_ approach, so score from each `must` or `should` will be added together to provide the final value of `_score`

You can combine `filters` inside `queries` and vice versa.

## Phrase query

Find documents where all terms are in the right order:

```json
{
  "query": {
    "match_phrase": {
      "title": "star wars"
    }
  }
}
```

In the index values are stored not only with the information in which document they exist but also where in the document they appear.

You can also say that you want to look for two word, and they might not appear one after another, you can allow other words between them:

```json
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "star beyond",
        "slop": 1
      }
    }
  }
}
```

The `slop` tells how you are willing to let a term move to satisfy a phrase (in either direction). For example `quick brown and red fox`
will match `quick fox` with a slop of 3. You can set s`slop`
to some big value and documents in which provided words are close together will have higher relevance.

## Pagination & Scroll

### Pagination

Pagination is based on `from` and `size` parameters. From values starts with `0`. Example:

```json
{
  "from": 2,
  "size": 2,
  "query": {
    "match": {
      "genre": "sci-fi"
    }
  }
}

```

Deep pagination can kill performance. Query still need to get every result and do sorting. You should set upper bounds limit for pagination.

### Scrolling

Behaves very similar to a cursor in a database query. Allows you to get even all data from the index. It is not longer recimended for deep
pagination.

To start with scrilling you will have to ask to open a scroll and set for how long it will live:

```
POST /movies/_search?scroll=1m
{
  "size": 2,
  "query": {
    "match": {
      "genre": "sci-fi"
    }
  }
}
```

Result will contain a `scroll_id` that can be used to retrieve the data:

```json
{
  "scroll": "1m",
  "scroll_id": "some-scroll-id"
}
```

Every scroll request will return `scroll_id`. It's value will sometimes change. It's best to always use the latest value. To get data using
scroll:

```
POST /_search/scroll
{
  "scroll": "1m",
  "scroll_id": "some-scroll-id"
}
```

You can also delete a scroll:

```
DELETE /_search/scroll
{
  "scroll_id" : "some-scroll-id"
}
```

### Search after

You can use search after to get next page of hits using set of search and sort values from the previous page. **It is recommended way of
accessing deep pagination.** If there is a refresh between requests, then order of results might change. You can create a
`point in time` to prevent this:

```
POST /movies/_pit?keep_alive=1m
```

will return `id` that can be used in search query:

```json
{
  "id": "some-id-1"
}
```

Next you can use the PIT id to start searching:

```
GET /_search
{
  "size": 2,
  "query": {
    "match": {
      "genre": "sci-fi"
    }
  },
  "pit": {
    "id": "some-id-1",
    "keep_alive": "1m"
  },
  "sort": [
    {
      "year": "asc"
    }
  ]
}
```

Please notice that request is against `_search` as index is described in PIT id. Result will contain new PIT id and `sort`. Using `sort`
value from last hit we can continue to get new elements. Example response:

```json
{
  "pit_id": "some-id-2",
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": null,
    "hits": [
      {
        "_index": "movies",
        "_id": "122886",
        "sort": [
          1420070400000
        ]
      }
    ]
  }
}
```

To get next batch of results:

```
GET /_search
{
  "size": 2,
  "query": {
    "match": {
      "genre": "sci-fi"
    }
  },
  "pit": {
    "id": "some-id-2",
    "keep_alive": "1m"
  },
  "sort": [
    {
      "year": "asc"
    }
  ],
  "search_after": [
    1420070400000
  ]
}
```

PIT should be removed after done searching:

```
DELETE http://localhost:9200/_pit

{
  "id": "some-id-1"
}
```

## Fuzzy search

Allows to search even when someone misspelled a world. Is using `levenshtein edit distance` to account for: substitutions, insertions or
deletions of character(s). By setting a distance you are telling now many characters can be changed.

Elastic search has `AUTO` setting for fuzzy search that takes into account length of the word and specifies levenshtein edit distance as:

- 0 - for words containing 1-2 characters,
- 1 - for words containing 3-5 characters,
- 2 - for anything else.

Example:

```json
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "intersteller", "fuzziness": 1
      }
    }
  }
}
```



