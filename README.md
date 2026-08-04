## About

This project was developed as part of a Master's thesis to satisfy the requirements for the degree of Master of Science in Electronics and ICT Engineering Technology at KULeuven.

# CeT2S-Agent

CeT2S-Agent is a context-enhanced text-to-SPARQL system built with FastAPI and LangGraph. It accepts a natural-language question and returns a SPARQL query for either DBpedia or Wikidata.

The agent is build upon and expands on the mKGQAgent developed by the WSE-research group at Leipzig University of Applied Sciences.
[mKGQAgent](https://github.com/WSE-research/text2sparql-agent)

## What this project does

The service runs a multi-step pipeline that can:
- translate the question to English,
- detect the expected answer type,
- retrieve in-context examples,
- build question-specific KG context,
- generate and refine a SPARQL query.

## Setup

1. Clone the repository.
2. Create and activate a virtual environment:

```bash
python -m venv venv
# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Set the required API key environment variable:

```bash
set CeT2SAgent_LLM=your_openrouter_api_key
```

On Linux/macOS, use `export CeT2SAgent_LLM=...` instead.

## Run the API

```bash
uvicorn main:app --reload
```

The service will be available at:

```text
http://localhost:8000
```

## API endpoint

### GET /CeT2SAgent

This endpoint converts a natural-language question into SPARQL.

### Query parameters

- `question` (required): the natural-language question to process
- `dataset` (optional): one of the supported datasets
  - `https://dbpedia.org/sparql`
  - `https://query.wikidata.org/sparql`
- `model_name` (optional): OpenRouter model identifier, for example `openai/gpt-4o-mini`
- `log_calls` (optional): enable/disable LLM call logging
- `temperature` (optional): LLM temperature, default `0`
- `use_translate` (optional): translate the question before processing
- `use_llm_translate` (optional): use an LLM for translation
- `use_icl` (optional): use in-context learning examples
- `use_eat` (optional): use expected-answer-type detection
- `use_context` (optional): use KG context generation

### Example request

```bash
curl "http://localhost:8000/CeT2SAgent?question=Who%20is%20the%20president%20of%20the%20United%20States%3F&dataset=https://dbpedia.org/sparql&model_name=openai/gpt-4o-mini"
```

### Example response

```json
{
  "dataset": "https://dbpedia.org/sparql",
  "question": "Who is the president of the United States?",
  "model_name": "openai/gpt-4o-mini",
  "translated_question": "Who is the president of the United States?",
  "query": "SELECT DISTINCT ?person WHERE { ... }",
  "prompt_tokens": 1234,
  "completion_tokens": 456,
  "requests": 6,
  "step_times": []
}
```

### Response fields

- `dataset`: the dataset URL used for the request
- `question`: the original user question
- `model_name`: the model identifier used
- `translated_question`: the translated or normalized question
- `query`: the generated SPARQL query
- `prompt_tokens`: total prompt tokens used
- `completion_tokens`: total completion tokens used
- `requests`: number of LLM requests made
- `step_times`: timing information for the pipeline steps

## Supported datasets

- `https://dbpedia.org/sparql`
- `https://query.wikidata.org/sparql`

## Notes

- The system depends on an OpenRouter-compatible API key via `CeT2SAgent_LLM`.
- If the dataset is unknown, the API returns an HTTP 404 error.
