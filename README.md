# Local Research OS: Autonomous RAG Agent 

A privacy-first, air-gapped AI system that autonomously monitors ArXiv for new research, ingests papers into a local Vector Database, and provides a citation-aware chat interface. Built to solve the "Context Loss" and "Source Hallucination" problems common in LLM retrieval.

## Architecture

## 🏗️ Architecture
```text
```mermaid
graph TD
    subgraph "External World"
        ArXiv[ArXiv API]
    end

    subgraph "The Scout (n8n)"
        Trigger_A[Daily Trigger] -->|Fetch| HTTP_A[HTTP Request]
        HTTP_A -->|Parse| Llama_Scout[Llama 3 (Analyst)]
        Llama_Scout -->|Score > 7| Downloader[Download PDF]
    end

    subgraph "Local File System"
        Downloader -->|Save| Folder[Downloads Folder]
    end

    subgraph "Ingestion Engine (n8n)"
        Folder -->|Watch| Trigger_B[Manual/Schedule Trigger]
        Trigger_B -->|Read| PDF_Read[Read Binary]
        PDF_Read -->|Split| Code_Node[JS Code: Split & Stamp]
        Code_Node -->|Vectorize| Embed_Model[Ollama: Nomic-Embed]
        Embed_Model -->|Store| Qdrant[(Qdrant DB)]
    end

    subgraph "The Brain (RAG)"
        User((User)) -->|Chat| WebUI[Open WebUI]
        WebUI -->|Query| Qdrant
        Qdrant -->|Context| Llama_Chat[Llama 3 (Chat)]
        Llama_Chat -->|Answer| WebUI
    end
    
    ArXiv --> HTTP_A
```

* **The Scout (Autonomous Agent):** Runs daily, queries ArXiv API, uses Llama 3 to score abstracts (1-10) for relevance, and auto-downloads high-value papers.
* **The Ingestion Engine (ETL Pipeline):** A batch-processing workflow in n8n that splits PDFs, injects "Source Stamps" into every text chunk (to guarantee citation accuracy), and embeds them into Qdrant.
* **The Brain (RAG Chat):** A local Llama 3.1 instance connected via Open WebUI that performs semantic search with a strict "Cite Sources" system prompt.

## Tech Stack

* **Orchestration:** n8n (Dockerized)
* **Vector Database:** Qdrant (Local)
* **Inference Engine:** Ollama (running Llama 3.1 8B)
* **Interface:** Open WebUI
* **Scripting:** JavaScript (Custom Node logic for metadata injection)

## Getting Started

### Prerequisites
* Docker Desktop (4GB+ RAM allocated)
* Ollama installed locally

### Installation
1.  Clone the repo:
    ```bash
    git clone [https://github.com/VS251/local-research-os.git](https://github.com/VS251/local-research-os.git)
    ```
2.  Start the stack:
    ```bash
    docker-compose up -d
    ```
3.  Import the workflows from the `/workflows` folder into n8n (localhost:5678).

## Key Features
* **Zero Data Ingress:** 100% local. No data is sent to OpenAI or Anthropic.
* **Source Stamping:** Custom ETL logic injects `SOURCE_FILENAME: {name}` into every vector embedding, solving the "Lost Header" problem in RAG retrieval.
* **Automatic Curation:** The system filters noise by "reading" abstracts for you before downloading.
