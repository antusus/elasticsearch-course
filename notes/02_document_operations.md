# Specifying document structure
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