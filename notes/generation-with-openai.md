[Previous](retrieval-with-minsearch.md) | [Next](searching-with-elasticsearch.md)

# Generation with LLM and RAG flow

## Overview

In this chapter, we will integrate the OpenAI API to generate answers based on the search results. We will build prompts and get responses from the OpenAI model. Instead of OpenAI API, we can use any other LLM.

## Topics Covered

- Introduction to OpenAI API
- Building Prompts
- Generating Answers

## Introduction to OpenAI API

OpenAI provides powerful APIs for natural language processing tasks.It can be used for various tasks such as generating text, summarizing information, translating languages, and answering questions.
We will use the OpenAI API to generate answers based on the search results from our indexed documents. We integrate prompts with RAG to enhance the quality of responses.

To use the OpenAI API, sign up for an account. After creating an account, navigate to the API section to receive an API key. Ensure your API key is kept secure and not shared publicly.

## Building Prompts

A prompt is the input you provide to the LLM to generate a response. It can be a question, a statement, or a set of instructions. The quality and clarity of your prompt directly affect the quality of the response. In general, building effective prompts is crucial for leveraging the full potential of LLMs.

To generate answers, we need to build prompts that provide context to the OpenAI model. Here is an example of how to build a prompt:

```python
from openai import OpenAI

client = OpenAI()

def build_prompt(query, search_results):
    prompt_template = """
    You're a course teaching assistant. Answer the QUESTION based on the CONTEXT from the FAQ database.
    Use only the facts from the CONTEXT when answering the QUESTION.

    QUESTION: {question}

    CONTEXT: 
    {context}
    """.strip()

    context = ""
    for doc in search_results:
        context += f"section: {doc['section']}\nquestion: {doc['question']}\nanswer: {doc['text']}\n\n"

    prompt = prompt_template.format(question=query, context=context).strip()
    return prompt
```

Here, the first step is to import the OpenAI library, which allows us to use its functions to interact with the language model.

Next, we define the `build_prompt`function. This function takes two arguments: `query` is the question we want to answer, and `search_results` is a list of documents retrieved from a search. 

The `prompt_template` specifies the format for the prompt. We will construct the final prompt using this template, incorporating the `context` string and and the given `query`.

We start by initializing an empty string `context`. This string will be filled with information from each document in the search results. For each document, we extract the `section`, `question`, and `text` (answer), format these details into a string, and append them to the context string.

Once the `context` is complete, we use the `format` method to insert the `query` and `context` into the `prompt_template`. This process creates the final prompt that will be sent to the OpenAI model.


## Generating Answers

Once we have the prompt, we can use the OpenAI API to generate answers.

```python
def llm(prompt):
    response = client.chat.completions.create(
        model='gpt-4o',
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content
```

Next, we execute the function with a sample query.

```python
query = 'how do I run kafka?'
search_results = index.search(query, filter_dict={'course': 'data-engineering-zoomcamp'}, boost_dict={'question': 3.0, 'section': 0.5}, num_results=5)
prompt = build_prompt(query, search_results)
answer = llm(prompt)
print(answer)
```

The `llm` function takes a single argument, `prompt`.
Inside the function, we use the `client.chat.completions.create` method to send the prompt to the OpenAI model specified by model='gpt-4o'. The messages parameter is a list containing a dictionary. The dictionary includes a `role` key set to `user` and a `content` key set to the `prompt`.
The function extracts the generated answer from the response object using `response.choices[0].message.content` and returns it.

Next, we will see how to replace the Minsearch with Elasticsearch.


[Previous](retrieval-with-minsearch.md) | [Next](searching-with-elasticsearch.md)