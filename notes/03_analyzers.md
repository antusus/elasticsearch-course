# Analyzers

When mapping field that contains text you can say in the index definition 
how are you going to use it. If you need only to do **exact match** or 
dealing with structured content (like: IDs, email, hostname, status code, 
zip codes or tags) you will use `keyword` mapping, and you will most likely 
use queries like `term`.

When you have unstructured text you might want to analyze it before inserting 
into an index. This when you will use `text` mapping and some analyzers. 
This will allow you to do partial matching or search with multiple terms where
document must not match all passed terms (score will determine how document 
matches the query).

Analyzers can make results to be: case-insensitive, stemmed, have stopwords
removed, synonyms applied, etc.