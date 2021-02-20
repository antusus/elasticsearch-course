# To Start
To run docker use:

`docker-compose up`

To stop:

`docker-compose down`

if you want to stop it and clear volume data use:

`docker-compose down -v`

By default ES is writting logs so they are accesible with 

`docker logs`

To change this and make it write to file you can set environment variable:

`ES_LOG_STYLE=file`

# Setting up elastic search data
Data was taken from: 
 - index definition: http://media.sundog-soft.com/es7/shakes-mapping.json

 - data: http://media.sundog-soft.com/es7/shakespeare_7.0.json

This describes data that we are going to put to ES. We can load index definition with:

`curl -H "Content-Type: application/json" -XPUT "localhost:9200/shakespeare" --data-binary @shake
s-mappings.json`

To load actual data:

`curl -H "Content-Type: application/json" -XPOST "localhost:9200/shakespeare/_bulk" --data-binary @shakespeare_7.0.json`

To test loaded data call:

`curl -H "Content-Type: application/json" -XGET "localhost:9200/shakespeare/_search?pretty" -d '{"query": { "match_phrase": {"text_entry": "to be or not to be"}}}'`
