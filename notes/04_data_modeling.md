# Data modeling

## Normalizing vs Denormalized data

Storage is cheap. Sometimes for sake of freedom to change data you want to have data normalized. For example let's say we can store movies
information in one index and ratings in another. If you want to get both you will need to execute two queries. If latency is a problem then
you might want to denormalize data. For example put movie title in each rating. It makes updating movie title very complicated but when
getting rating you will get title immediately. Besides, move titles are not changing a lot.

## Parent / Child Relationships

We would like to introduce franchises like "Star Wars" or "Star Trek" to moves. We will define `series` index:

```json
{
  "mappings": {
    "properties": {
      "movie_to_franchise": {
        "type": "join",
        "relations": {
          "franchise": "movie"
        }
      }
    }
  }
}
```

If we look for entries that are in the "Star Wars" franchise:
```json
{
  "query": {
    "has_parent": {
      "parent_type": "franchise",
      "query": {
        "match": {
          "title": "Star Wars"
        }
      }
    }
  }
}
```
entry can look like this (some fields are omitted):

```json
{
  "_index": "series",
  "_id": "5378",
  "_source": {
    "id": "5378",
    "film_to_franchise": {
      "name": "film",
      "parent": "1"
    },
    "title": "Star Wars: Episode II - Attack of the Clones",
    "year": "2002"
  }
}
```

### Parent-join restrictions
 - Only one join field mapping is allowed per index.
 - Parent and child documents must be indexed on the same shard. This means that the same routing value needs to be provided when getting, deleting, or updating a child document.
 - An element can have multiple children but only one parent.
 - It is possible to add a new relation to an existing join field.
 - It is also possible to add a child to an existing element but only if the element is already a parent.

