[Previous](02-semantic-search-with-elasticsearch.md) | [Next](04-advanced-semantic-search.md)


# Evaluating Retrieval

### Why Evaluate Retrieval?

Evaluating retrieval performance is crucial to ensure that your search system returns relevant and accurate results. It helps in identifying areas for improvement and optimizing the search algorithms.

We will explore the importance of gold standard data sets and the role of evaluation metrics in determining the best approach for various datasets.


Here are the main points:

1. **Different Ways to Find Stuff**: There are many methods to store and get data, like Min Search, Elastic Search, and Vector Search. Each has its own strengths, depending on what you need.
2. **Checking If It Works**: To know which method is best, you need to test them using special techniques. This is like having a way to say, "For this search, these results are the best."
3. **Gold Standard Data**: You need a "gold standard" or the best possible example of search results. This means knowing the best answers to common questions in advance.
4. **Measuring Performance**: You use different scores to see how good the search results are. These scores help you understand if the search engine is showing the right results.
5. **Creating Test Data with LLMs**: Sometimes, you can use Large Language Models (LLMs) to create these "best example" data sets if you don't have them already.
6. **Fine-Tuning**: You can adjust different settings in tools like Elastic Search to improve the search results.


### Gold Standard Data Sets

A critical aspect of evaluation is the use of gold standard data sets. These are ground truth data sets where the relevant documents for each query are known. For instance, given a query like "Can I still join the course?", the relevant documents would be pre-identified. This allows for a clear benchmark to assess the performance of different retrieval methods.


### Generating Ground Truth Data

To check if a search system is working well, we need a set of good examples of questions and their correct answers. This helps us see if the system finds the right answers.

Typically, we have many queries and their corresponding relevant documents or one query has multiple relevant documents, helping to measure the accuracy of search results.

For practical purposes, we generate a data set where each query has one relevant document. This simplification helps in experimenting with and evaluating different retrieval techniques.

### Preparing the documents

#### Generating Stable IDs for Documents

To accurately track relevant documents, each document is assigned a unique ID. By maintaining consistent IDs, we can manage changes and updates to the document set without affecting the evaluation process. This helps know which answer goes with which question.

Here are the steps to generate stable IDs:

1. **Concatenate Document Attributes**: Combine key attributes of the document (e.g., course name, question, and a portion of the text) into a single string. This ensures that the ID is unique to the specific content of the document.
2. **Generate MD5 Hash**: Use the MD5 hashing algorithm to create a hash from the concatenated string. MD5 is chosen for its balance of speed and uniqueness.
3. **Extract a Substring of the Hash**: To keep the ID concise, extract a substring (e.g., the first 8 characters) of the MD5 hash. This substring serves as the document's unique ID.
4. **Assign the ID to the Document**: Attach the generated ID to the document. This ID will be used to reference the document in the evaluation process.

#### Human Annotation

In production systems, human annotators and domain experts review documents and queries to identify relevant document-query pairs. Although this method is time-consuming, it results in high-quality data, often referred to as "gold standard" data. Observing user interactions and evaluating system responses also contribute to refining the data set.

#### Using Large Language Models (LLMs)

LLMs can be used to generate user queries based on FAQ records. By creating multiple questions for each FAQ record, we ensure that the generated questions are varied and relevant. This automated approach speeds up the process of creating a ground truth data set and is suitable for initial experiments before deploying a production system.

```python
prompt_template = """
You emulate a student who's taking our course.
Formulate 5 questions this student might ask based on a FAQ record. The record
should contain the answer to the questions, and the questions should be complete and not too short.
If possible, use as fewer words as possible from the record. 

The record:

section: {section}
question: {question}
answer: {text}

Provide the output in parsable JSON without using code blocks:

["question1", "question2", ..., "question5"]
""".strip()
```


#### Creating the Ground Truth Dataset

We send the generated prompt to OpenAI's API and receive a JSON response containing the generated questions. These responses are stored in a dictionary with the document ID as the key. The parsed JSON responses are then processed to extract the questions and associate them with their respective courses and document IDs. This structured data is saved in a CSV file for further analysis. The CSV file format makes it easy to manipulate and use the data in various evaluation tools and techniques.

### Evaluation Metrics

Using the generated data set, we can evaluate different search systems by computing metrics such as Hit Rate and Mean Reciprocal Rank (MRR). These metrics help us understand how well the search system retrieves relevant documents and identify areas for improvement.


- Hit Rate: measures how often the search system manages to find a relevant document for a query.
For example, if you have 10 queries and the search system finds a relevant document for 8 of those queries, the Hit Rate is 8/10 or 80%.

- MRR: Let's break it down

Reciprocal Rank is the inverse of the rank at which the first relevant document is found.
For example, if the first relevant document is found at rank 1, the reciprocal rank is 1/1 = 1. If it’s found at rank 2, the reciprocal rank is 1/2 = 0.5, at rank 3 it is 1/3 ≈ 0.33, and so on.

So Mean Reciprocal Rank is the average of the reciprocal ranks across multiple queries.
It evaluates the rank position of the first relevant document across different queries.
For example, if you have three queries and their reciprocal ranks are 1, 0.5, and 0.33, the MRR would be (1 + 0.5 + 0.33) / 3 ≈ 0.61.

More info on metrics: https://github.com/DataTalksClub/llm-zoomcamp/blob/main/03-vector-search/eval/evaluation-metrics.md


### Evaluating Text Search Overview

We assess the performance of Elastic Search and Min Search.

For Elastic Search we:

- Load documents from a JSON file (`documents-with-ids.json`) and initializes an Elastic Search client connected to a local instance.

- Define the settings and mappings for Elastic Search index. The loaded documents are then indexed into Elastic Search.


```python
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
            "course": {"type": "keyword"},
            "id": {"type": "keyword"},
        }
    }
}

index_name = "course-questions"

es_client.indices.delete(index=index_name, ignore_unavailable=True)
es_client.indices.create(index=index_name, body=index_settings)

for doc in tqdm(documents):
    es_client.index(index=index_name, document=doc)
```

- Perform a search query on the Elastic Search index. The search query uses a `multi_match` query to search across multiple fields (`question`, `text`, `section`) with different weights. It filters the search results by the specified course and returns the top 5 search results.

```python
def evaluate_text_search(query, course):
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
                        "course": course
                    }
                }
            }
        }
    }

    response = es_client.search(index=index_name, body=search_query)
    result_docs = [hit['_source'] for hit in response['hits']['hits']]
    return result_docs
```

- Load ground truth data from CSV file (`ground-truth-data.csv`) and convert it into a list of dictionaries.

- For each query in the ground truth data, perform a search using the `elastic_search` function, compare the search results with the ground truth document ID, and store the relevance results.

- Calculate evaluation metrics:  Hit rate and MRR

```python
def hit_rate(relevance_total):
    cnt = 0

    for line in relevance_total:
        if True in line:
            cnt = cnt + 1

    return cnt / len(relevance_total)

def mrr(relevance_total):
    total_score = 0.0

    for line in relevance_total:
        for rank in range(len(line)):
            if line[rank] == True:
                total_score = total_score + 1 / (rank + 1)

    return total_score / len(relevance_total)
```

Similarly, we apply the above steps for Min Search. In the end, we print and compare the evaluation metrics between Elastic Search and Min Search.

## Application and Tools for Fine-Tuning

Various tools and configurations can be applied to optimize search retrieval systems. Elastic Search, for example, allows for numerous adjustments such as boosting coefficients and field inclusion, enabling fine-tuning of the search results. Evaluating these configurations against a gold standard data set helps in identifying the most effective approach.



In the next chapter, we will delve into advanced semantic search techniques.

[Previous](02-semantic-search-with-elasticsearch.md) | [Next](04-advanced-semantic-search.md)
