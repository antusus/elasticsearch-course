# Sorting

A text field that is analyzed for search cannot be used for sorting documents. 
It exists in an inverted index as individual words not as an entire string.

If you want to use text field for sorting you have to create not analyzed copy. 
For example we want to make moves sortable by `title`:
```json
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "raw": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```