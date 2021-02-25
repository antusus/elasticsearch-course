# Inserting documents

## Specifying document structure

You can tell ES how to treat data inside a document by providing mappings. For example assume we have a document called `movies` and you
want to have year field to be treated as a date:

```json
{
  "mappings": {
    "properties": {
      "year": {
        "type": "date"
      }
    }
  }
}
```

There are lots of types to choose from. You can also specify if a field should be indexed or not: `"index": "not_analyzed"`.

## Analyzers

Additionally, you can define an analyzer that can manipulate data before adding it to index.

Analyzers can:

- filter characters, like remove HTML encoding, convert & to and,
- tokenize values on white spaces, strings or non letters
- filter tokens, apply lowercase, stemming, synonyms apply stopwords and more

# Concurrency

Elastic Search is distributed. Operations that are changing documents must replicate new document  
version to other nodes in a cluster. Elastic Search is also concurrent and asynchronous. Those replications can happen in parallel and out
of sequence. There must be a way to ensure that older version of document will never override newer one.

To do that every operation performed on a document is assigned a sequence number by the primary shard that coordinates that change. Sequence
number is increased with each operation. New operation will always have higher sequence number than older ones. Elastic Search uses sequence
number to make sure that newer document version will not be overridden by a change that has smaller number.

Each document has `_version` field that describes... version of a document.
Any change on the document will increase version. 

Each operation on a document is also changing two fields that are uniquely identifying
a change:
- primary term: `_primary_term`,
- sequence number: `_seq_no`. 


You can use them to make sure you are updating a document in a proper version by adding: `if_seq_no` and
`if_primary_term` to request. Assume we want to change a document that when we read it 
had `_primary_term=1` and `_seq_no=24`:

```
PUT movies/_doc/1567?if_seq_no=24&if_primary_term=1
{
  "title": "Tomorrow Never Dies",
  "genre": ["Action", "Drama"],
  "year": 1997
}
```

If the primary term and sequence number matches update will be successful, otherwise it will return an error.

You can deal with that with an update API, and use `retry_on_conflict` parameter, for example:
```
POST movies/_update/1567?retry_on_conflict=5
{
  "script": {
    "source": "if(!ctx._source.genre.contains(params.genre)) {ctx._source.genre.add(params.genre)}",
    "params": {
      "genre": "Comedy"
    }
  }
}
```
Will retry up to 5 times if conflict occurs. We are interested only to add this genre
to a document.

# Updating document

In ES documents are immutable. They have _version field. To update document you need to 
upload it with never version. An old document is marked for deletion and will later 
be removed.

You can upload partial of a document to update only some fields (using POST REST API), 
or you can update whole document (using PUT REST API).

You can update only one document or use query and update many of them. When updating one 
document you can apply optimistic concurrency mechanism with `if_primary_term`
and `if_seq_no` parameters.

When updating a document you can specify a query and all documents matching query will 
be updated, or you can update all documents.

Update is done in batches. First search return batch of documents to change and next
batch update is done by ES until there are no more documents matching query. 

You can specify how many shards must accept copy before proceeding with the request, 
you can set this up with `wait_for_active_shards` control.

# Deleting document

Use DELETE REST API call and document will be marked for deletion.

```
DELETE /movies/_doc/1224
```

Delete operation can have optimistic concurrency control enabled with `if_primary_term`
and `if_seq_no` parameters passed in delete request.

You can specify a version of document to delete. After removal version is still available
for short period of time to allow for concurrent operations. You can specify amount
of time by `index.gc_deletes` setting (defaults to 60s).
