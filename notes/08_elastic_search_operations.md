# Choosing right number of shards

An index is split into shards. Shard may be on a different node in a cluster. Shared is self-contained Lucene index. When inserting
documents into an index they are being hashed into particular shard.

Remember that you can add replica shards later, but you **can't** add master shards. Shard is quite expensive in resources as it is a Lucene
index.

There is no good formula to calculate amount of shard needed. The bast way to decide how many shards you need is to play with the data.
Start small and try breaking it. Then you can add more shards and play again.

# Using multiple indices as scaling strategy

Assume you do not want/can't reindex. You can add a new index to hold new data. You can use both indices 
during search and use `index aliases` to make this easy. With time based data, you can have one index 
per time frame (like a day or month). You then can have aliases `logs_current`, `logs_last_3_months`
and point those aliases to specific indices as they rotate. Example:

```
POST /_aliases
{
  "actions": [
    {
      "add": {"alias": "logs_current", "index": "logs_2021_02"}
    },
    {
      "remove": { "alias": "logs_current", "index": "logs_2021_01" }
    },
    {
      "add": { "alias": "logs_last_3_months", "index": "logs_2021_01" }
    },
    {
      "remove": { "alias": "logs_last_3_months", "index": "logs_2020_12" }
    }
  ]
}
```

**Alias can join multiple indices.**

# Index lifecycle management

1. Hot - index is actively updated and queried,
1. Warm - index is actively queried but not updated,
1. Cold - index is not updated and not frequently queried,
1. Delete - index is deleted.

Transitions between those steps can be defined in ILM and can determine when:
 - Spin up a new index when an index reaches a certain size or number of documents
 - Create a new index each day, week, or month and archive previous ones
 - Delete stale indices to enforce data retention standards

After creating a lifecycle policy you can set it up in the index you are creating.

# Choosing cluster hardware

RAM is the bottleneck. Elasticsearch suggests 64GB of ram per machine (32GB for Elastic search and 
32GB for OS and cache for lucene). You should not run ES using less than 8GB. You do not want more 
than 32GB because memory pointers are getting to big and performance may suffer.

With memory there is a question about `Heap space`. Default 1GB maybe to small for production:
 - Half or less of you physical memory should be used for elastic search. If you are not doing 
aggregation on analyzed fields you can specify less memory for Elastic search
 - Other half of memory will be consumed by OS and lucene cache
 - Smaller heap results in faster garbage collection and more memory for cache

You would like fast SSD disks. You can use RAID0 becasue replication is embeded in cluster.
You do not need fast CPU. You will need fast network.

