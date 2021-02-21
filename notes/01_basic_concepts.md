# Basics
## Document
 - Something you are search in for in ES
 - Must have ID that you provide or make ES to generate for you

## Indices
 - The Highest level entity that you can query
 - Index defines document schema
 - Can return list of documents that you were querying for
 - Indices are inverted, in short they keep terms (elements of documents) with reference to document they appear in

## Relevance
 - **TF-IDF** - means Term Frequency * Inverse Document Frequency
 - **Term Frequency** - describes how often a term appears in _a given document_
 - **Document Frequency** - describes how often term appears in _all documents_
 - **Term Frequency / Document Frequency** - measures relevance of a term in a document

# Scaling
## Shards
 - Indices are split to shards
 - Shard in an instance of [Lucene index](https://lucene.apache.org/)
 - Shards may be stored on different nodes in a cluster. If you do that you can add more machines that will help distribute load
 - There is a function that can tell in which shard document exists

## Primary and Replica Shards
 - Index will have at least one primary shard and several replicas
 - Write requests are routed to master shard and then replicated
 - Application should round-robin requests among nodes
 - Read requests are routed to any master or replica shard
 - When master fails ES will elect new master among existing replicas
 - You can execute write requests to any node. You will be redirected to a node containing primary shard that will accept operation
 - **You CANNOT change number of primary shards in your index later**
 - **You CAN add more read replicas later**
 - Worst case scenario you can re-index your data and new index will have better write throughput
 - You can specify number of shards with a PUT command via REST, command below will create 3 primary shards
each with 1 replica which will lead to 6 shards in total
```
PUT /someindex 
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}
```


