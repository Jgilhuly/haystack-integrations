---
layout: integration
name: Arize AI
description: Monitor and trace your Haystack pipelines Arize and Phoenix
authors:
    - name: Arize AI
      socials:
        github: Arize-ai
        twitter: ArizePhoenix
        linkedin: arizeai
pypi: https://pypi.org/project/openinference-instrumentation-haystack/
repo: https://github.com/Arize-ai/openinference
type: Monitoring Tool
report_issue: https://github.com/Arize-ai/openinference/issues
logo: /logos/arize.png
version: Haystack 2.0
toc: true
---
### **Table of Contents**
- [Overview](#overview)
- [Features](#features)
- [Installation](#installation)
- [Use Arize Phoenix](#usage)

## Overview

Arize is an AI Observability and Evaluation platform designed to help you troubleshoot, evaluate, and experiment on LLM and ML applications. Developers use Arize to get applications working quickly, evaluate performance, detect and prevent production issues, and curate datasets.

**Phoenix** is Arize's open-source library that offers developers the quickest way to troubleshoot, evaluate, and experiment with LLM applications.

You can bring your Haystack pipelines to both Arize and Arize Phoenix (the OSS offering). Both integrations use the same underlying library, `openinference-instrumentation-haystack`.

- [Documentation for Arize AI + Haystack](https://docs.arize.com/arize/large-language-models/tracing/auto-instrumentation/haystack)
- [Documentation for Phoenix + Haystack](https://docs.arize.com/phoenix/tracing/integrations-tracing/haystack)

## Installation

```bash
pip install openinference-instrumentation-haystack haystack-ai arize-otel opentelemetry-sdk opentelemetry-exporter-otlp
```

## Use Arize Phoenix
### Setting up the `HaystackInstrumentor`

To trace any Haystack pipeline with Phoenix, you have to create a `HaystackInstrumentor`. Haystack pipelines that run within the same environment send traces to Phoenix.
You have 2 options:
- The easiest option is to [launch a hosted phoenix instance](https://phoenix.arize.com/) and set your Phoenix API key with the `PHOENIX_API_KEY` environment variable
- There is also the option to self-host Phoenix if you prefer, [here](https://docs.arize.com/phoenix/deployment/deploying-phoenix)

```python
from arize_otel import register_otel, Endpoints
from openinference.instrumentation.haystack import HaystackInstrumentor

# Setup OTEL tracing for hosted Phoenix
register_otel(
    endpoints=[Endpoints.HOSTED_PHOENIX],
    api_key="YOUR_HOSTED_PHOENIX_API_KEY"
)

HaystackInstrumentor().instrument()
```

Now, you can run a Haystack pipeline within the same environment, resulting in the following trace:

![Arize Demo](https://raw.githubusercontent.com/deepset-ai/haystack-integrations/main/images/arize-demo.gif)
> To run the example below, export your OpenAI Key to the `OPENAI_API_KEY` environment variable.
```python 
import os
from haystack import Pipeline, Document
from haystack.utils import Secret
from haystack.document_stores.in_memory import InMemoryDocumentStore
from haystack.components.retrievers.in_memory import InMemoryBM25Retriever
from haystack.components.generators import OpenAIGenerator
from haystack.components.builders.answer_builder import AnswerBuilder
from haystack.components.builders.prompt_builder import PromptBuilder

document_store = InMemoryDocumentStore()
document_store.write_documents([
    Document(content="My name is Jean and I live in Paris."), 
    Document(content="My name is Mark and I live in Berlin."), 
    Document(content="My name is Giorgio and I live in Rome.")
])

prompt_template = """
Given these documents, answer the question.
Documents:
{% for doc in documents %}
    {{ doc.content }}
{% endfor %}
Question: {{question}}
Answer:
"""

retriever = InMemoryBM25Retriever(document_store=document_store)
prompt_builder = PromptBuilder(template=prompt_template)
llm = OpenAIGenerator()

rag_pipeline = Pipeline()
rag_pipeline.add_component("retriever", retriever)
rag_pipeline.add_component("prompt_builder", prompt_builder)
rag_pipeline.add_component("llm", llm)
rag_pipeline.connect("retriever", "prompt_builder.documents")
rag_pipeline.connect("prompt_builder", "llm")

question = "Who lives in Paris?"
results = rag_pipeline.run(
    {
        "retriever": {"query": question},
        "prompt_builder": {"question": question},
    }
)

```