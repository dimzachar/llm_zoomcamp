# Polars Tutorial with Ollama

This tutorial demonstrates how to use the Polars library for data manipulation and analysis, and how to integrate it with Ollama for generating insights.

You can start Ollama with docker (see https://github.com/DataTalksClub/llm-zoomcamp/blob/main/02-open-source/README.md)


```
docker run -it \
    -v ollama:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```

and pull the `phi3` model 
```
docker exec -it ollama bash
ollama pull phi3
```

## Setup

First, we need to import the necessary libraries:

```python
import polars as pl
from datetime import datetime
```

## Creating a DataFrame

We will create a simple DataFrame with some financial data:

```python
data = {
    "date": [datetime(2024, 7, 1), datetime(2024, 7, 2), datetime(2024, 7, 3), datetime(2024, 7, 4), datetime(2024, 7, 5)],
    "revenue": [100, 150, 200, 250, 300],
    "expenses": [80, 120, 160, 200, 240]
}

df = pl.DataFrame(data)
print(df)
```

This will output:

```
shape: (5, 3)
┌─────────────────────┬─────────┬──────────┐
│ date                ┆ revenue ┆ expenses │
│ ---                 ┆ ---     ┆ ---      │
│ datetime[μs]        ┆ i64     ┆ i64      │
╞═════════════════════╪═════════╪══════════╡
│ 2024-07-01 00:00:00 ┆ 100     ┆ 80       │
│ 2024-07-02 00:00:00 ┆ 150     ┆ 120      │
│ 2024-07-03 00:00:00 ┆ 200     ┆ 160      │
│ 2024-07-04 00:00:00 ┆ 250     ┆ 200      │
│ 2024-07-05 00:00:00 ┆ 300     ┆ 240      │
└─────────────────────┴─────────┴──────────┘
```

## Filtering Data

Next, we will filter the DataFrame to include only the rows where the  `date` is between July 2, 2024, and July 4, 2024:

```python
filtered_df = df.filter(
    pl.col("date").is_between(datetime(2024, 7, 2), datetime(2024, 7, 4))
)
print(filtered_df)
```

We use the `pl.col` function to select the `date` column and the `is_between` method to specify the date range. The `filter` method returns a new DataFrame `filtered_df` that contains only the rows that meet the condition. We then print the filtered DataFrame:

```
shape: (3, 3)
┌─────────────────────┬─────────┬──────────┐
│ date                ┆ revenue ┆ expenses │
│ ---                 ┆ ---     ┆ ---      │
│ datetime[μs]        ┆ i64     ┆ i64      │
╞═════════════════════╪═════════╪══════════╡
│ 2024-07-02 00:00:00 ┆ 150     ┆ 120      │
│ 2024-07-03 00:00:00 ┆ 200     ┆ 160      │
│ 2024-07-04 00:00:00 ┆ 250     ┆ 200      │
└─────────────────────┴─────────┴──────────┘
```

## Converting to Pandas DataFrame

We can convert the filtered Polars DataFrame to a Pandas DataFrame:

```python
df_pandas = filtered_df.to_pandas()
```

- Polars for Efficiency: Use Polars for data loading, manipulation, and filtering due to its performance advantages.
- Conversion to Pandas: Convert to Pandas when you need to use libraries or APIs that require Pandas DataFrames, such as the OpenAI API.


## Using Ollama for Analysis

We will use Ollama to analyze the financial data and predict future values. First, we need to set up the Ollama client:

```python
from openai import OpenAI

client = OpenAI(
    base_url='http://localhost:11434/v1/',
    api_key='ollama',
)
```

### Building the Prompt

We will build a prompt to send to Ollama:

```python
def build_prompt(df):
    prompt = "Analyze the financial data:\\n"
    for row in df.itertuples(index=True):
        prompt += f"Date: {row.date.strftime('%Y-%m-%d')}, Revenue: ${row.revenue}, Expenses: ${row.expenses}\\n"
    prompt += "Predict the revenue and expenses for the next date."
    return prompt
```

### Sending the Prompt to Ollama

We will send the prompt to Ollama and get the response:

```python
def llm(prompt):
    response = client.chat.completions.create(
        model='phi3',
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content
```

### Getting the Prediction

Finally, we will build the prompt, send it to Ollama, and print the prediction:

```python
prompt = build_prompt(df_pandas)
answer = llm(prompt)
print(answer)
```

This will output a prediction based on the provided financial data.

## Conclusion

In this tutorial, we demonstrated how to use Polars for data manipulation and analysis, and how to integrate it with Ollama for generating insights. Polars provides a powerful and efficient way to handle data, and Ollama can be used to generate meaningful predictions and analyses.


## Resources & extra tools

- [An Introduction to Polars: Python's Tool for Large-Scale Data Analysis](https://www.datacamp.com/blog/an-introduction-to-polars-python-s-tool-for-large-scale-data-analysis)
- [Polars AI GitHub Repository](https://github.com/wiseaidev/polars-ai)
- [An Introduction to Pandas AI](https://www.datacamp.com/blog/an-introduction-to-pandas-ai)
- [Pandas AI Documentation](https://docs.pandas-ai.com/library)
