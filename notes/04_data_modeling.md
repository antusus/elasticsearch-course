# Data modeling

## Dynamic vs Explicit Mapping

In **Dynamic mapping** Elastic Search creates new fields automatically after indexing document. It's useful when you are experimenting with
data. You can use dynamic templates to define custom mappings.

In **Explicit mapping** you can define mapping definition:

- tell which fields should be treated as full text fields, tell which fields to ignore, or treat as keywords
- tell which fields contain numbers, dates or geolocations,
- specify dates format
- create rules to control the mapping for dynamically added fields.

Dynamic mapping can lead to **Mapping Explosion** which cal lead to poor performance, out of memory errors. You can try to limit that with
Explicit mapping but even then there is nothing stopping you to add too many new fields. There is a way to set **mapping limit settings** to
limit number of fields that can be created and prevent mapping explosion.

## There is a hard limit of 1000 fields in a document

You can change it with `index.mapping.total_fields.limit` setting, but it may lead to performance 
degradations and memory issues.

## Normalizing vs Denormalized data

Storage is cheap. Sometimes for sake of freedom to change data you want to have data normalized. For example let's say we can store movies
information in one index and ratings in another. If you want to get both you will need to execute two queries. If latency is a problem then
you might want to denormalize data. For example put movie title in each rating. It makes updating movie title very complicated but when
getting rating you will get title immediately. Besides, move titles are not changing a lot.

## Flattened data type

Assume we are storing logs in Elastic Search cluster, and we have host property that might have lots of different inner fields for example:

 ```json
{
  "host": {
    "name": "App-01",
    "ip": "123.43.12.42"
  }
}
```

```json
{
  "host": {
    "name": "App-02",
    "ip": "123.43.12.56",
    "os": "linux",
    "arch": "arm"
  }
}
```

By default, ES would add specified mappings to include add the inner fields of the `host` field. If we do not want that we can specify thr
mapping of `host` field to be **flattened**:

```json
{
  "properties": {
    "host": {
      "type": "flattened"
    }
  }
}
```

This will cause ES not to map anny inner fields of host. It reduces size of mapping. However, this means that the `host` field will be
treated as keyword and will not be analyzed. We can ask for documents with
`host.arch = "arm"`:

```json
{
  "query": {
    "match": {
      "host.arch": "arm"
    }
  }
}
```

this will do an exact match on inner parts of field `host`.

Supported queries on `flattened` data type:

- term, terms and terms_set
- prefix
- range (non-numerical range operations)
- match and multi_match (but we have to provide explicit keywords)
- query_string and simple_query_string
- exists

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
- Parent and child documents must be indexed on the same shard. This means that the same routing value needs to be provided when getting,
  deleting, or updating a child document.
- An element can have multiple children but only one parent.
- It is possible to add a new relation to an existing join field.
- It is also possible to add a child to an existing element but only if the element is already a parent.

## Exception mappings

Assume that we have a field that we have mapped as integer:

```json
{
  "mappings": {
    "properties": {
      "port": {
        "type": "number"
      }
    }
  }
}
```

When inserting new documents port value will be mapped to integer so values 8080 and "9200" are correct. If a value that is not parsable
occurs, the exception will be thrown. We can ask ES to ignore malformed data but to do that we need to change the index.

```
PUT /my-index/_settings

{ "index.mapping.ignore_malformed": true }
```

malformed fields will not be indexed and will be placed in `ignored` section of a document. 
`ignore_malformed` will not handle malformed JSON values. Those will cause exception to be thrown.

## Closing and opening index

When you need to perform changes on the index you can close it `POST /my-index/_close`. All read/write operations are blocked. This allows
closed indices not to have maintain internal data structures for indexing and searching documents. This also reduces overhead on the
cluster.

If you perform any changes to the index upon opening it master will restart index shards to reflect new state of the index. Shards will go
through recovery process. Data of opened/closed indices are automatically replicated by the cluster.

You can open and close multiple indices at once.

Closed indices consume a significant amount of disk-space. You can disable closing indices.



