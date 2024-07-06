# Ollama

## Introduction

Ollama is a versatile tool that allows you to run large language models (LLMs) on a CPU. This tutorial will guide you through the steps to set up and use Ollama, including running it with Docker, downloading models, and integrating it with Elasticsearch.

## Video

[![Ollama - Running LLMs on a CPU](https://markdown-videos-api.jorgenkh.no/youtube/PVpBGs_iSjY)](https://www.youtube.com/watch?v=PVpBGs_iSjY&list=PL3MmuxUbc_hIB4fSqLy_0AfTjVLpgjV3R)

## Prerequisites

Before you start, ensure you have the following installed on your system:

1. **Docker**: Follow the [official Docker installation guide](https://docs.docker.com/get-docker/) to install Docker on your system.
2. **Python**: Ensure you have Python 3.7 or later installed. You can download it from the [official Python website](https://www.python.org/downloads/).

## Steps

### 1. Running Ollama with Docker

Docker allows you to run applications in isolated environments called containers. To run Ollama with Docker, follow these steps:

1. **Pull the Ollama Docker Image**: Open your terminal and run the following command to pull the Ollama Docker image from Docker Hub:

```bash
docker pull ollama/ollama
```

2. **Run the Ollama Container**: Execute the following command to run the Ollama container:

```bash
docker run -it \
    --rm \
    -v ollama:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```

3. **Check the Ollama Version**: To find the version of the Ollama client, enter the container and execute:

```bash
docker exec -it ollama ollama -v
```

### 2. Downloading an LLM

To download a large language model (LLM) using Ollama, follow these steps:

1. **Enter the Container**: If you are not already inside the container, enter it using:

```bash
docker exec -it ollama bash
```

2. **Download the Model**: Run the following command inside the Docker container to download the `gemma:2b` model:

```bash
ollama pull gemma:2b
```

Alternatively, if running from docker-compose, you can use:

```bash
ollama pull gemma:2b
```

### 3. Running the LLM

To test the downloaded model, follow these steps:

1. **Create a Python Script**: Create a Python script named `test_llm.py` with the following content:

```python
from openai import OpenAI

client = OpenAI(
    base_url='http://localhost:11434/v1/',
    api_key='ollama',
)

prompt = "10 * 10"
response = client.chat.completions.create(
    model='gemma:2b',
    messages=[{"role": "user", "content": prompt}]
)
print(response.choices[0].message.content)
```

2. **Run the Script**: Execute the script to test the model:

```bash
python test_llm.py
```

### 4. Downloading the Weights

To avoid downloading the model weights every time you run the container, follow these steps:

1. **Create a Local Directory**: Create a local directory to store the model weights:

```bash
mkdir ollama_files
```

2. **Run the Container with Volume Mapping**: Run the container with the local directory mapped to the container's `/root/.ollama` directory:

```bash
docker run -it --rm -v ./ollama_files:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

3. **Download the Model**: Enter the container and download the model:

```bash
docker exec -it ollama bash
ollama pull gemma:2b
```


### 5. Adding the Weights

To create a new Docker image with the downloaded weights, follow these steps:

1. **Create a Dockerfile**: Create a file named `Dockerfile` with the following content:

```dockerfile
FROM ollama/ollama
COPY ./ollama_files /root/.ollama
```

2. **Build the Docker Image**: Build the new Docker image:

```bash
docker build -t ollama-gemma2b .
```

### 6. Serving the Model

To serve the model and test it, follow these steps:

1. **Run the New Docker Image**: Run the new Docker image:

```bash
docker run -it --rm -p 11434:11434 ollama-gemma2b
```

2. **Test the Model**: Create a Python script named `test_model.py` with the following content:

```python
from openai import OpenAI

client = OpenAI(
    base_url='http://localhost:11434/v1/',
    api_key='ollama',
)

prompt = "What's the formula for energy?"
response = client.chat.completions.create(
    model='gemma:2b',
    messages=[{"role": "user", "content": prompt}],
    temperature=0.0
)
print(response.choices[0].message.content)

completion_tokens = response.usage.completion_tokens
print(f"Number of completion tokens: {completion_tokens}")
```


### 7. Integrating with Elasticsearch

To integrate Ollama with Elasticsearch using Docker-Compose, follow these steps:

1. **Create a Docker-Compose File**: Create a file named `docker-compose.yaml` with the following content:

```yaml
version: '3.8'

services:
    elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.4.3
    container_name: elasticsearch
    environment:
        - discovery.type=single-node
        - xpack.security.enabled=false
    ports:
        - "9200:9200"
        - "9300:9300"

    ollama:
    image: ollama/ollama
    container_name: ollama
    volumes:
        - ollama:/root/.ollama
    ports:
        - "11434:11434"

volumes:
    ollama:
```

2. **Run Docker-Compose**: Execute the following command to start the services:

```bash
docker-compose up
```

3. **Re-run the Module 1 Notebook**: Ensure everything is set up correctly by re-running the module 1 notebook.

## Conclusion

By following these steps, you can set up and use Ollama to run LLMs on a CPU, download models, and integrate with Elasticsearch.
