[Previous](preparing-the-environment.md) | [Next](generation-with-openai.md)


# Retrieval with MinSearch

## Overview

In this chapter, we will build a simple search engine using the `minsearch` library. We will index the documents and perform searches. Check also

* Video: https://www.youtube.com/watch?v=nMrGK5QgPVE
* Code: https://github.com/alexeygrigorev/build-your-own-search-engine
* Code: https://github.com/DataTalksClub/llm-zoomcamp/blob/main/01-intro/minsearch.py

for the implementation of the search engine.

## Topics Covered

- Introduction to MinSearch
- Indexing Documents
- Performing Searches

## Introduction to MinSearch

MinSearch is a simple search engine library that allows you to index and search documents using TF-IDF and cosine similarity.

Efficient document retrieval is crucial in various applications, from search engines to recommendation systems. MinSearch provides a lightweight and easy-to-use solution for these tasks.

## Understanding TF-IDF and Cosine Similarity

TF-IDF stands for Term Frequency-Inverse Document Frequency. It is a numerical statistic that reflects the importance of a word in a document relative to a collection of documents (corpus).

Cosine similarity measures the cosine of the angle between two vectors, which in the context of document retrieval, represents the similarity between two documents.

## Indexing Documents

MinSearch requires documents to be indexed before searching. This process involves calculating the TF-IDF vectors for each document.

We will load the documents from a JSON file. Each document should have fields that we want to index, such as `question`, `text`, and `section`. Next, we initialize the MinSearch index and fit it with the loaded documents.

```python
import minsearch
import json

with open('documents.json', 'rt') as f_in:
    docs_raw = json.load(f_in)

documents = []

for course_dict in docs_raw:
    for doc in course_dict['documents']:
        doc['course'] = course_dict['course']
        documents.append(doc)

index = minsearch.Index(
    text_fields=["question", "text", "section"],
    keyword_fields=["course"]
)

index.fit(documents)
```

where
- `text_fields` are fields containing the main textual content of the documents.
- `keyword_fields` are fields containing keywords for filtering, such as course names.

## Performing Searches

Once the documents are indexed, we can perform searches using the `search` method.
We start by defining the query we want to search for.

```python
query = 'the course has already started, can I still enroll?'
```
We perform the search by specifying the query and additional parameters like filtering and boosting.

```python
results = index.search(
    query, 
    filter_dict={'course': 'data-engineering-zoomcamp'}, 
    boost_dict={'question': 3.0, 'section': 0.5}, 
    num_results=5
)

print(results)
```

where

- `filter_dict` is used to filter the search results based on keyword fields.
- `boost_dict` is used to give higher importance to certain fields during the search.
- `num_results` specifies the number of top results to return.


The `search` method returns a list of results, each containing the matched document and its relevance score.

```python
for result in results:
    print(f"Document: {result['document']}, Score: {result['score']}")
```

[Previous](preparing-the-environment.md) | [Next](generation-with-openai.md)