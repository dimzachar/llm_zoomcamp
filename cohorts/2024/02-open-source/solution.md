## Homework: Open-Source LLMs

### Q1. Running Ollama with Docker

Let's run ollama with Docker. We will need to execute the 
same command as in the lectures:

```bash
docker run -it \
    --rm \
    -v ollama:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```

Then, 

```bash
docker exec -it ollama bash
```

inside the container

```bash
ollama -v
```

**Answer:** The version of the Ollama client is `0.1.48`.

### Q2. Downloading an LLM

To download the `gemma:2b` model

```bash
docker exec -it ollama ollama pull gemma:2b
```

or just

```bash
ollama pull gemma:2b`
```

when running from docker-compose.

Check the metadata in `/.ollama/models/manifests/registry.ollama.ai/library/gemma`, use `cat 2b`

**Answer:** The content of the file related to `gemma` is:
```json
{"schemaVersion":2,"mediaType":"application/vnd.docker.distribution.manifest.v2+json","config":{"mediaType":"application/vnd.docker.container.image.v1+json","digest":"sha256:887433b89a901c156f7e6944442f3c9e57f3c55d6ed52042cbb7303aea994290","size":483},"layers":[{"mediaType":"application/vnd.ollama.image.model","digest":"sha256:c1864a5eb19305c40519da12cc543519e48a0697ecd30e15d5ac228644957d12","size":1678447520},{"mediaType":"application/vnd.ollama.image.license","digest":"sha256:097a36493f718248845233af1d3fefe7a303f864fae13bc31a3a9704229378ca","size":8433},{"mediaType":"application/vnd.ollama.image.template","digest":"sha256:109037bec39c0becc8221222ae23557559bc594290945a2c4221ab4f303b8871","size":136},{"mediaType":"application/vnd.ollama.image.params","digest":"sha256:22a838ceb7fb22755a3b0ae9b4eadde629d19be1f651f73efb8c6b4e2cd0eea0","size":84}]}
```

### Q3. Running the LLM

Test the following prompt: "10 * 10"

Run
```python
ollama run gemma:2b
```


or in a python file
```python
prompt = "10 * 10"
response = client.chat.completions.create(
    model='gemma:2b',
    messages=[{"role": "user", "content": prompt}]
)
print(response.choices[0].message.content)
```

**Answer:** Sure, here is the answer to the question:

10 * 10 = 100.

### Q4. Downloading the weights

Map the `/root/.ollama` folder to a local directory and check the size of the `ollama_files/models` folder.

```bash
mkdir ollama_files
docker run -it --rm -v ./ollama_files:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
docker exec -it ollama ollama pull gemma:2b
du -h ollama_files/models
```


**Answer:** The size of the `ollama_files/models` folder is `1.6G` (1.7G is option on course).
```bash
du -h ollama_files/models
8.0K    ollama_files/models/manifests/registry.ollama.ai/library/gemma
12K     ollama_files/models/manifests/registry.ollama.ai/library
16K     ollama_files/models/manifests/registry.ollama.ai
20K     ollama_files/models/manifests
1.6G    ollama_files/models/blobs
1.6G 
```


### Q5. Adding the weights

Create a `Dockerfile` to add the weights to a new image

```dockerfile
FROM ollama/ollama
COPY ./ollama_files /root/.ollama
```

**Answer:** The `COPY` command is `COPY ./ollama_files /root/.ollama`.

### Q6. Serving it

Build and run the new Docker image and test the prompt "What's the formula for energy?" with `temperature=0.0`

```bash
docker build -t ollama-gemma2b .
docker run -it --rm -p 11434:11434 ollama-gemma2b
```

Test the prompt:
```python
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

**Answer:** The number of completion tokens is `304`.
