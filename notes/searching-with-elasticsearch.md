[Previous](generation-with-openai.md)

# Searching with Elasticsearch

## Overview

In this chapter, we will replace the custom search engine with Elasticsearch for more robust search capabilities. We will set up Elasticsearch, index the documents, and perform searches.

Elasticsearch was initially released in 2010 by Shay Banon. It was built on top of Apache Lucene, a popular open-source search library. Over the years, Elasticsearch has grown in popularity and capabilities, becoming a core component of the Elastic Stack (formerly known as the ELK Stack), which includes Logstash, Kibana, and Beats.

https://www.elastic.co/elasticsearch

Features:

- **Scalability**: Elasticsearch can scale horizontally to handle large volumes of data across multiple nodes.
- **Real-time Search**: Provides near real-time search capabilities, enabling quick and efficient data retrieval.
- **Distributed Architecture**: Ensures high availability and fault tolerance.
- **Flexible Schema**: Supports dynamic mapping and schema-less document indexing.
- **RESTful API**: Offers a simple and powerful API for interacting with Elasticsearch.


## Topics Covered

- Setting Up Elasticsearch
- Indexing Documents
- Performing Searches

## Setting Up Elasticsearch

Install the ElasticSearch client for Python:
    ```bash
    pip install elasticsearch
    ```

### Running Elasticsearch with Docker

Run Elastic Search 8.4.3 using Docker:

```bash
docker run -it \
    --rm \
    --name elasticsearch \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.4.3
```

use `-m 4GB \` flag to specify the memory limit for the Docker container.


Use the GET /_cluster/health endpoint to get detailed information about the cluster health, including the status, number of nodes, and number of active shards and the cluster information by running:
    ```bash
    curl -X GET "localhost:9200/_cluster/health?pretty"
    curl localhost:9200
    ```

### Remove the Existing Container

If you get conflict errors, you might need to remove (or rename) that container to be able to reuse that name

```bash
docker stop elasticsearch
docker rm elasticsearch
   ```

## Indexing Documents

To index the documents in Elasticsearch, we need to define the index settings and mappings, and then index each document.

### Index Settings and Mappings

First, create an instance of the Elasticsearch client on Jupyter, which connects to the Elasticsearch server. Index settings include configurations such as the number of shards and replicas. Shards help distribute the data and load across the cluster, while replicas provide redundancy for fault tolerance.

We are interested in Mappings that define the structure of the documents within an index. Each field in the documents can have specific data types and properties.

```python
from elasticsearch import Elasticsearch

es_client = Elasticsearch('http://localhost:9200')

index_settings = {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "text": {"type": "text"},
            "section": {"type": "text"},
            "question": {"type": "text"},
            "course": {"type": "keyword"} 
        }
    }
}

index_name = "course-questions"
es_client.indices.create(index=index_name, body=index_settings)

for doc in documents:
    es_client.index(index=index_name, document=doc)
```

We use the "keyword" type for a field like "course" when we want to filter documents based on the course name or run aggregations to count the number of documents per course.

## Performing Searches

Once the documents are indexed, we can perform searches using Elasticsearch. This section explains how to construct and execute search queries to retrieve relevant documents from the index.

### Search Query

The following example demonstrates how to create a search query in Elasticsearch using the Python client.

```python
query = 'I just discovered the course. Can I still join it?'

def elastic_search(query):
    search_query = {
        "size": 5,
        "query": {
            "bool": {
                "must": {
                    "multi_match": {
                        "query": query,
                        "fields": ["question^3", "text", "section"],
                        "type": "best_fields"
                    }
                },
                "filter": {
                    "term": {
                        "course": "data-engineering-zoomcamp"
                    }
                }
            }
        }
    }

    response = es_client.search(index=index_name, body=search_query)
    result_docs = [hit['_source'] for hit in response['hits']['hits']]
    return result_docs

search_results = elastic_search(query)
print(search_results)
```

The search query in Elasticsearch is composed of several key components that work together to retrieve relevant documents. 

- The `size` parameter limits the number of search results to 5, ensuring that the output remains manageable and focused. 
- The `multi_match` query searches across multiple fields—specifically `question`, `text`, and `section`—and boosts the relevance of the `question` field with a factor of 3, making matches in this field more significant. 
- The `query` option holds the user's search input, while the `fields` option lists the fields to search in, applying boosting as defined. 
- The `type` parameter of the `multi_match` query is set to `best_fields`, which finds the single best matching field.
- A `filter` is applied to restrict the search results to documents where the `course` field equals `data-engineering-zoomcamp`. 


This combination of components ensures that the search results are both relevant and precise, with a specific focus on certain fields and criteria.


### Integrating Elasticsearch with OpenAI API

Finally, we integrate Elasticsearch with the OpenAI API to generate answers based on the search results.

```python
search_results = elastic_search(query)
prompt = build_prompt(query, search_results)
answer = llm(prompt)
print(answer)
```

By integrating Elasticsearch, we can handle larger and more complex datasets, providing a robust and scalable solution for generating answers with the OpenAI API.


![alt text](image-4.png)


[Previous](generation-with-openai.md)