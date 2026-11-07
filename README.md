# Text-to-SQL Agent
## Natural Language Interface for Relational Databases

The Text-to-SQL LangGraph Agent is an open-source AI workflow that converts natural-language questions into SQL queries and understandable answers. It runs locally through Ollama and open-source language models, without depending on function calling.

## Overview

This project makes relational databases accessible to users who may not know SQL. A user can ask a question in plain English—for example, “How many customers are from the USA?”—and the system generates the corresponding SQL statement, executes it against the database, and returns both the query results and a conversational explanation.

There are two implementations:

1. **LangChain notebook:** `text_to_SQL_Langchain_agent.ipynb` uses LangChain’s built-in SQL agent.
2. **Custom LangGraph workflow:** A sequence of nodes and custom tools provides deterministic SQL generation and execution.

## System Design

The application uses a modular, stateful design built around LangGraph, Ollama, and SQLite.

### Agent orchestration

- Conversation state is managed through `MessagesState`.
- The LangGraph workflow contains five sequential nodes and deterministic routing from `START` to `END`.
- Custom `@tool` decorators connect the agent to database operations.

### Language model configuration

| Component | Configuration |
|---|---|
| Model | CodeGemma 7B Instruct (Q4_K_S), configurable |
| Inference runtime | Ollama at `http://localhost:11434` |
| Prompt context | Structured prompts with schema injection and response formatting |

### Database configuration

| Component | Configuration |
|---|---|
| Database engine | SQLite |
| Database interface | LangChain `SQLDatabase` abstraction |
| Example schema | Chinook music database containing 11 tables, configurable |
| Query safeguards | Read-only operations enforced through query validation |

## Capabilities

### Natural-language querying
The agent translates questions into SQL, accounts for table relationships, and supports queries involving multiple-table joins.

### Multiple ways to interact
The project can be used through:
- A Streamlit web application for interactive use.
- FastAPI REST endpoints for integration with other systems.
- A Jupyter Notebook for development and experimentation.

### Query safeguards and error handling
Database operations are restricted to read-only access. The workflow includes query validation, error handling, and automatic result limits to reduce misuse and prevent excessive queries.

### Explanatory results
Responses include the generated SQL statement, the raw database output, and a natural-language explanation to make the results easier to interpret.

## Repository Layout

```text
langgraph-text-to-sql-agent/
├── configuration.py
├── FastAPI.py
├── langgraphagent.py
├── streamlitapp.py
├── text_to_SQL_Langchain_agent.ipynb
├── README.md
└── License
```

- `configuration.py` — environment setup and configuration.
- `FastAPI.py` — API layer for backend integration.
- `langgraphagent.py` — core LangGraph workflow and agent logic.
- `streamlitapp.py` — Streamlit frontend for user interaction.
- `text_to_SQL_Langchain_agent.ipynb` — notebook implementation using the LangChain agent.

## Example Applications

| Use case | How the project can be used |
|---|---|
| Data exploration | Explore relational data by asking questions without writing SQL. |
| Business intelligence | Obtain quick insights from structured databases to support decisions. |
| Developer integrations | Connect the FastAPI endpoint to analytics dashboards or internal data systems. |
| Education and training | Demonstrate the conversion of natural-language questions into SQL. |
| AI agent integrations | Provide database access to chatbots and virtual assistants. |

## Demo

### Streamlit interface

<img width="1915" height="1080" alt="Streamlit interface" src="https://github.com/user-attachments/assets/93324b05-b375-4cff-a76f-c6a2cf9a2175" />

### FastAPI response

<img width="1718" height="878" alt="FastAPI response" src="https://github.com/user-attachments/assets/2a500b25-98ee-4743-baed-c7a76f5c35d1" />

## Installation and Configuration

### Requirements

- Python 3.10 or later.
- A CUDA-compatible GPU is optional and can improve processing speed.
- 8 GB or more of RAM is recommended.

### 1. Get the repository

```bash
git clone https://github.com/adhmara-prani/Text-2-SQL-agent.git
cd langgraph-text-to-sql-agent
```

### 2. Install Python dependencies

```bash
pip install -r requirements.txt
```

### 3. Install and prepare Ollama

Install Ollama:

```bash
curl -fsSL https://ollama.ai/install.sh | sh
```

Download the model used by the agent:

```bash
ollama pull codegemma:7b-instruct-v1.1-q4_K_S
```

### 4. Set the local configuration

In `configuration.py`, update the model and database paths to match your environment:

```python
model_name = "codegemma:7b-instruct-v1.1-q4_K_S"
database_name = "sqlite:///Chinook.db"
```

## Running the Application

### Start the API

Run the FastAPI server:

```bash
python FastAPI.py
```

Once started, the API is available at:

`http://localhost:8000`

### Open the web interface

In a separate terminal, launch Streamlit:

```bash
streamlit run streamlit_app.py
```

The interface will be available at:

`http://localhost:8501`

## Request Lifecycle

A typical request moves through the following stages:

1. **Question submission:** The user enters a natural-language question, such as “How many customers are from the USA?”
2. **Table discovery:** The application retrieves the available table names from the SQLite database.
3. **Schema retrieval:** The schema for the relevant tables is collected and supplied as context to the language model.
4. **SQL generation:** CodeGemma 7B, running through Ollama, generates the SQL query.
5. **Query execution:** The query is validated and executed under read-only restrictions.
6. **Response construction:** The application returns the generated SQL, raw database results, and a natural-language explanation of the findings.

The LangGraph workflow follows this sequence:

```text
START
  |
  v
list_tables
  |
  v
get_schema
  |
  v
generate_sql
  |
  v
run_query
  |
  v
generate_response
  |
  v
END
```

## Technology Stack

| Technology | Role in the project | Documentation |
|---|---|---|
| LangChain | Framework for building LLM-driven applications | [LangChain](https://python.langchain.com) |
| LangGraph | Node-based orchestration for LLM workflows | [LangGraph](https://github.com/langchain-ai/langgraph) |
| Ollama | Local language-model inference engine | [Ollama](https://ollama.ai) |
| CodeGemma 7B (Q4_K_S) | Quantized, instruction-tuned model for code understanding | [Gemma Models](https://ai.google.dev/gemma) |
| SQLite | Lightweight relational database for structured data | [SQLite](https://sqlite.org) |
| FastAPI | Python framework for building high-performance APIs | [FastAPI](https://fastapi.tiangolo.com) |
| Streamlit | Framework for interactive data applications | [Streamlit](https://streamlit.io) |
