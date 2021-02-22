# Inserting documents
## Specifying document structure
You can tell ES how to treat data inside a document by providing mappings.
For example assume we have a document called `movies` and you want
to have year field to be treated as a date:
```json
{
  "mappings": {
    "properties": {
      "year": {"type":  "date"}
    }
  }
}
```
There are lots of types to choose from. You can also specify if a 
field should be indexed or not: `"index": "not_analyzed"`.

## Analyzers
Additionally, you can define an analyzer that can manipulate data before
adding it to index. 

Analyzers can:
 - filter characters, like remove HTML encoding, convert & to and,
 - tokenize values on white spaces, strings or non letters
 - filter tokens, apply lowercase, stemming, synonyms apply stopwords and more 

# Updating document
In ES documents are immutable. They have _version field. To update document you need to upload it with never version.
An old document is marked for deletion and will later be removed.

You can upload partial of a document to update only some fields (using POST REST API),
or you can update whole docment (using PUT REST API).

# Deleting document
Use DELETE REST API call and document will be marked for deletion. 
