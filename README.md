## Docker Compose
Each folder in this project has its own independent docker-compose project.
There are multiple versions of compose files because it was a progression in getting the final one working.
In addition, playing with other user's compose files.

 

### docker-compose up/down
* elk - elasticsearch 2 with kibana 4 and logstash 2
* elk7 - elasticsearch 7 with kibana 7 without logstash
* elk7withlogstash - multi-node elasticsearch 7 with kibana 7 with logstash 7
* nodes1_elasticsearch_logstash_7 - single node elasticsearch 7 with kibana 7 with logstash 7
* nodes3_elasticsearch_logstash_7 - multi-node elasticsearch 7 with kibana 7 with logstash 7
* kafka2_elasticsearch_logstash_7 - kafka 2, logstash 7 and a single node of elasticsearch 7 

## Elasticsearch
### Check cluster details
http://localhost:9200

```
{
  "name" : "es01",
  "cluster_name" : "es-docker-cluster",
  "cluster_uuid" : "hnq4VXqLSJa4NJLHzWcU2Q",
  "version" : {
    "number" : "7.12.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "3186837139b9c6b6d23c3200870651f10d3343b7",
    "build_date" : "2021-04-20T20:56:39.040728659Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
### Check cluster
    http://localhost:9200
Results
```
{
  "name" : "es01",
  "cluster_name" : "es-docker-cluster",
  "cluster_uuid" : "B8iMZ78bRoqwFOIy-W7Yeg",
  "version" : {
    "number" : "7.12.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "3186837139b9c6b6d23c3200870651f10d3343b7",
    "build_date" : "2021-04-20T20:56:39.040728659Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
### Check elastic search nodes

    http://localhost:9200/_cat/nodes?v&pretty

Results

    ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
    172.29.0.2           61          57   1    0.00    0.11     0.25 cdfhilmrstw *      es01

### Check elastic shards and topics
"The shards command is the detailed view of what nodes contain which shards. It will tell you if it’s a primary or replica, the number of docs, the bytes it takes on disk, and the node where it’s located."

    http://localhost:9200/_cat/shards
Results
```
.ds-ilm-history-5-2021.05.05-000001 0 p STARTED             192.168.112.2 es01
logstash-2021.05.05-000001          0 p STARTED    7 30.9kb 192.168.112.2 es01
logstash-2021.05.05-000001          0 r UNASSIGNED                        
```
logstash-2021.05.05-000001 is a topic that is related to heartbeat.

# Logstash & Elastic Search

[Configuring Logstash for Docker](https://www.elastic.co/guide/en/logstash/current/docker-config.html)

[Configuring Elasticsearch for Docker](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/docker.html)

To test setup initial setup, one can use the heartbeat by placing the following in the pipeline config.
File location in setups pipeline/logstash.conf then start with docker compose.
```
input {
  # Listens on 514/udp and 514/tcp by default; change that to non-privileged port
  syslog { port => 51415 }
  # Default port is 12201/udp
  gelf { }
  # This generates one test event per minute.
  # It is great for debugging, but you might
  # want to remove it in production.
  heartbeat { }
}

output {
  elasticsearch {
    hosts => ["es01:9200"]
  }
  # This will output every message on stdout.
  # It is great when testing your setup, but in
  # production, it will probably cause problems;
  # either by filling up your disks, or worse,
  # by creating logging loops! BEWARE!
  stdout {
    codec => rubydebug
  }
}
```
Check results using _cat/shards to look up topic and then make query

    http://localhost:9200/logstash-2021.05.05-000001/_search?pretty
    
Results

```
{
  "took" : 72,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 15,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "logstash-2021.05.05-000001",
        "_type" : "_doc",
        "_id" : "_uMhPXkBfZ2q0Shwd1JM",
        "_score" : 1.0,
        "_source" : {
          "host" : "62c5c663b2d5",
          "@version" : "1",
          "@timestamp" : "2021-05-05T15:24:33.889Z",
          "message" : "ok"
        }
      },
      ...................................
```

For more Info on 
[Compact and aligned text (CAT) APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html)

## Kafka
[Configuring Kafka Docker](https://github.com/wurstmeister/kafka-docker)

## Docker Commands
Friendly reminders

    docker-compose up
    docker-compose down 
    docker network prune
    docker volume prune    



## References
