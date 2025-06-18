---
title: How to Create a Self-Hosted Chatbot That Knows Your Docs
date: 2025-06-17T13:05:16.383Z
description: "Learn to build a private, self-hosted chatbot for your internal documentation. This guide covers open-source LLMs (Ollama), vector databases (ChromaDB), and Retrieval-Augmented Generation (RAG) using LangChain, with full code examples."
tags: [LLM, RAG, Python, Self-Hosted, AI, Chatbot, Documentation, Ollama, LangChain, ChromaDB]
categories: [AI/ML, DevOps, Development]
comments: true
---

The dream of a chatbot that truly understands your company's internal documentation, project specifications, or extensive knowledge base is no longer a fantasy. While powerful, publicly hosted large language models (LLMs) like ChatGPT are often a non-starter for sensitive internal data due to privacy concerns and their inherent lack of knowledge about your specific, proprietary information.

This is where self-hosting, combined with a technique called Retrieval-Augmented Generation (RAG), becomes incredibly powerful. You get the intelligence of an LLM, the privacy of your own infrastructure, and the contextual awareness of your private documents.

In this detailed guide, we'll build a self-hosted chatbot from the ground up, designed to answer questions based on your custom documentation. We'll prioritize clear, pragmatic steps with working code examples, focusing on common open-source tools.

Let's get started.

## Core Components of Our Doc Bot

Before diving into the code, let's understand the key players involved:

1.  **Large Language Model (LLM)**: This is the "brain" of our chatbot, responsible for generating human-like text responses. For self-hosting, we'll use open-source models run locally.
2.  **Embedding Model**: An embedding model converts text into numerical representations (vectors). These vectors capture the semantic meaning of the text, allowing us to find "similar" pieces of text even if they don't share exact keywords.
3.  **Vector Database**: This specialized database stores the numerical embeddings of your documents. It's optimized for fast similarity searches, which is crucial for RAG.
4.  **Retrieval-Augmented Generation (RAG)**: This is the magic. When a user asks a question, we first "retrieve" relevant snippets from your documents using the vector database. These snippets, along with the user's question, are then fed to the LLM, allowing it to generate an informed answer grounded in your data.
5.  **Orchestration Framework**: To glue all these components together smoothly, we'll use a framework like LangChain, which simplifies the process of building complex LLM applications.

For our practical example, we'll use:
*   **LLM Runtime**: [Ollama](https://ollama.ai/) for easy local LLM deployment.
*   **LLM Model**: A smaller, efficient model like `llama2` or `mistral` runnable via Ollama.
*   **Embedding Model**: `all-MiniLM-L6-v2` from Hugging Face (via `sentence-transformers`).
*   **Vector Database**: [ChromaDB](https://www.trychroma.com/), a lightweight and easy-to-use option that can run in-memory or persist data.
*   **Orchestration**: [LangChain](https://www.langchain.com/langchain).

## Step 1: Setting Up Your Environment

First, ensure you have Python (3.8+) and `pip` installed. We'll use a virtual environment for a clean setup.

```bash
# Create a project directory
mkdir self-hosted-doc-bot
cd self-hosted-doc-bot

# Create a virtual environment
python3 -m venv venv

# Activate the virtual environment
# On Linux/macOS:
source venv/bin/activate
# On Windows:
# venv\Scripts\activate

# Install required Python packages
pip install langchain langchain-community ollama chromadb

# Note: langchain-community pulls in sentence-transformers as a dependency
# when you use HuggingFaceEmbeddings, so you usually don't need to install
# sentence-transformers explicitly. But it's good to be aware.
```

## Step 2: Preparing Your Documentation

Our chatbot needs documents to learn from. For this example, let's create a simple `docs` directory with a few `.txt` files. In a real scenario, these could be PDFs, Markdown files, web pages, etc. LangChain supports many document loaders.

Create a directory named `docs` and add some sample files:

```bash
mkdir docs
```

`docs/project_scope.txt`:
```
Project Name: Phoenix AI
Objective: Develop an AI assistant to automate customer support inquiries, specifically focusing on technical documentation lookup for our software products.
Key Features:
- Natural language processing for query understanding.
- Integration with existing knowledge base (Confluence, Jira).
- Scalable to handle 1000+ concurrent users.
- Deployment target: On-premise Kubernetes cluster.
Timeline: Q1 2024 - Q3 2024
Team Lead: Dr. Anya Sharma
```

`docs/troubleshooting_guide.txt`:
```
Troubleshooting Guide for Phoenix AI Installation:
1. Check System Requirements: Ensure your server meets minimum CPU (8 cores), RAM (32GB), and storage (200GB SSD) requirements.
2. Network Connectivity: Verify that ports 80, 443, and 5000 are open and accessible.
3. Database Connection: Confirm that the PostgreSQL database is running and accessible from the Phoenix AI application server. Check `connection_string` in `config.yaml`.
4. Log Files: Examine `/var/log/phoenix-ai/app.log` and `/var/log/phoenix-ai/db.log` for error messages.
If issues persist, contact support at support@phoenixai.com.
```

### Loading and Splitting Documents

Documents are often too large to fit into an LLM's context window. We need to split them into smaller, semantically meaningful chunks. LangChain's `RecursiveCharacterTextSplitter` is excellent for this.

Create a Python script named `ingest_docs.py`:

```python
# ingest_docs.py
import os
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_community.vectorstores import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Define the directory containing your documents
DOCS_DIR = "docs"
# Define the directory for the Chroma vector store
CHROMA_DB_DIR = "chroma_db"

def load_documents(directory):
    """Loads all text files from a given directory."""
    documents = []
    for filename in os.listdir(directory):
        if filename.endswith(".txt"):
            filepath = os.path.join(directory, filename)
            loader = TextLoader(filepath, encoding="utf-8")
            documents.extend(loader.load())
            print(f"Loaded {filepath}")
    return documents

def split_documents(documents):
    """Splits documents into smaller, manageable chunks."""
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=1000,      # Max characters in a chunk
        chunk_overlap=200,    # Overlap between chunks to maintain context
        length_function=len,
        is_separator_regex=False,
    )
    chunks = text_splitter.split_documents(documents)
    print(f"Split {len(documents)} documents into {len(chunks)} chunks.")
    return chunks

def create_vector_store(chunks, db_dir):
    """Creates embeddings and stores them in a Chroma vector database."""
    # Note: Using a common sentence transformer model.
    # It will be downloaded the first time it's used.
    print("Initializing embedding model...")
    embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")

    print(f"Creating Chroma vector store at {db_dir}...")
    # Chroma.from_documents will create embeddings and add them to the store
    vector_store = Chroma.from_documents(
        documents=chunks,
        embedding=embeddings,
        persist_directory=db_dir
    )
    vector_store.persist()
    print("Vector store created and persisted successfully.")
    return vector_store

if __name__ == "__main__":
    print("Starting document ingestion process...")
    
    # 1. Load documents
    loaded_documents = load_documents(DOCS_DIR)
    if not loaded_documents:
        print(f"No documents found in {DOCS_DIR}. Please add some .txt files.")
    else:
        # 2. Split documents
        document_chunks = split_documents(loaded_documents)
        
        # 3. Create and persist vector store
        create_vector_store(document_chunks, CHROMA_DB_DIR)
    
    print("Document ingestion complete.")
```

Run the ingestion script:

```bash
python ingest_docs.py
```

Sample output:

```output
Starting document ingestion process...
Loaded docs/project_scope.txt
Loaded docs/troubleshooting_guide.txt
Split 2 documents into 4 chunks.
Initializing embedding model...
.
. (This might download the model, showing progress)
.
Creating Chroma vector store at chroma_db/...
Vector store created and persisted successfully.
Document ingestion complete.
```

You should now see a `chroma_db` directory created in your project, containing the persisted vector embeddings.

## Step 3: Setting Up Your Self-Hosted LLM (Ollama)

Ollama makes it incredibly easy to download, run, and interact with open-source LLMs locally.

### 3.1 Install Ollama

Follow the instructions on the [Ollama website](https://ollama.ai/download) to install it for your operating system (Linux, macOS, Windows). It typically involves a single command or a simple installer.

For Linux (example):

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### 3.2 Download an LLM

Once Ollama is installed, you can download a model. We'll use `llama2` for this example, but you can choose others like `mistral`, `gemma`, etc., depending on your system's resources and preference.

```bash
ollama pull llama2
```

Sample output:

```output
pulling manifest
pulling 00fdd3427dfa... 100%|██████████| 3.8 GB/3.8 GB [00m00s, 71.2 MB/s]
pulling 8a34d0b135c3... 100%|██████████| 12 KB/12 KB [00m00s, 7.3 MB/s]
pulling d4695be5d367... 100%|██████████| 44 B/44 B [00m00s, 19.8 MB/s]
pulling f0f400713b13... 100%|██████████| 44 B/44 B [00m00s, 22.8 MB/s]
pulling 0c58971f163e... 100%|██████████| 4.8 KB/4.8 KB [00m00s, 3.2 MB/s]
verifying sha256 digest
setting permissions for 00fdd3427dfa...
.
.
success
```

### 3.3 Test Ollama API

Ollama runs a local server (by default on `http://localhost:11434`) that provides an API. Let's test it with `curl`:

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama2",
  "prompt": "Tell me a joke.",
  "stream": false
}'
```

Sample output:

```output
{"model":"llama2","created_at":"2023-10-27T10:30:00.123456789Z","response":"Why don't scientists trust atoms?\n\nBecause they make up everything!","done":true,"context":[...]}
```

If you get a response, your local LLM is running!

## Step 4: Building the RAG Chain

Now, let's connect everything: the user's query, the vector store (for retrieval), and the LLM (for generation).

Create a Python script named `chatbot.py`:

```python
# chatbot.py
from langchain_community.llms import Ollama
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_community.vectorstores import Chroma
from langchain.chains import RetrievalQA

# Define the directory for the Chroma vector store
CHROMA_DB_DIR = "chroma_db"

def create_rag_chain():
    """Initializes and returns the RAG chain."""
    print("Loading embedding model...")
    embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")

    print(f"Loading Chroma vector store from {CHROMA_DB_DIR}...")
    vector_store = Chroma(
        persist_directory=CHROMA_DB_DIR,
        embedding_function=embeddings
    )

    # Note: Use a more specific model if you pulled one (e.g., "llama2", "mistral")
    # Ensure Ollama server is running and the model is pulled.
    print("Initializing Ollama LLM...")
    llm = Ollama(model="llama2") # Make sure "llama2" is pulled via `ollama pull llama2`

    # Create a retriever from the vector store
    # k specifies the number of relevant document chunks to retrieve
    retriever = vector_store.as_retriever(search_kwargs={"k": 2})

    # Create the RAG chain
    # The chain will:
    # 1. Take the user query.
    # 2. Use the retriever to find relevant chunks from the vector store.
    # 3. Pass the query and the retrieved chunks to the LLM to generate an answer.
    print("Creating RetrievalQA chain...")
    qa_chain = RetrievalQA.from_chain_type(
        llm=llm,
        chain_type="stuff", # 'stuff' puts all retrieved docs into the prompt
        retriever=retriever,
        return_source_documents=True # Optional: returns the docs used for generation
    )
    print("RAG chain created.")
    return qa_chain

if __name__ == "__main__":
    qa_bot = create_rag_chain()

    print("\nChatbot is ready! Type 'exit' to quit.")
    while True:
        user_query = input("You: ")
        if user_query.lower() == 'exit':
            print("Exiting chatbot. Goodbye!")
            break

        print("Thinking...")
        response = qa_bot.invoke({"query": user_query})

        print("\nBot:")
        print(response["result"])
        # print("\nSource Documents:")
        # for doc in response["source_documents"]:
        #     print(f"- {doc.metadata['source']}")
        #     print(f"  {doc.page_content[:150]}...") # Print first 150 chars of content
        # print("-" * 50)
```

Now, run your chatbot script:

```bash
python chatbot.py
```

Sample interaction:

```output
Loading embedding model...
Loading Chroma vector store from chroma_db/...
Initializing Ollama LLM...
Creating RetrievalQA chain...
RAG chain created.

Chatbot is ready! Type 'exit' to quit.
You: What is the Phoenix AI project about?
Thinking...

Bot:
The Phoenix AI project aims to develop an AI assistant to automate customer support inquiries, specifically focusing on technical documentation lookup for software products. Key features include natural language processing, integration with existing knowledge bases, scalability for over 1000 concurrent users, and deployment on an on-premise Kubernetes cluster. The project timeline is Q1 2024 - Q3 2024, and the team lead is Dr. Anya Sharma.

You: How can I troubleshoot network issues during installation?
Thinking...

Bot:
To troubleshoot network connectivity issues during Phoenix AI installation, you should verify that ports 80, 443, and 5000 are open and accessible. This is part of ensuring that the application can communicate correctly within your environment.

You: What are the minimum RAM requirements?
Thinking...

Bot:
The minimum RAM requirement for the Phoenix AI server is 32GB.

You: Tell me a joke.
Thinking...

Bot:
I'm sorry, as an AI assistant focusing on technical documentation, I don't have jokes in my knowledge base. How can I help you with your documentation questions?

You: exit
Exiting chatbot. Goodbye!
```

Notice how the bot correctly answered questions based *only* on the documentation we provided. When asked for a joke (which wasn't in the docs), it intelligently declined or gave a generic LLM response, demonstrating it's grounding its answers in the provided context.

## Self-Hosting Considerations and Production Readiness

Building a local proof-of-concept is one thing; deploying a robust, performant, and secure self-hosted chatbot is another.

### Hardware Requirements

*   **LLM (Ollama)**: This is the most resource-intensive component.
    *   **RAM**: You need enough RAM to load the model weights. A 7B parameter model (like `llama2`) might require 8GB-16GB of RAM. Larger models (13B, 70B) will need significantly more (e.g., 20GB+ for 13B, 80GB+ for 70B).
    *   **GPU**: While Ollama can run models on CPU, a powerful GPU (NVIDIA with CUDA or AMD with ROCm) will dramatically speed up inference. Models are often "quantized" (e.g., Q4_K_M) to reduce their size and memory footprint, making them more runnable on consumer GPUs.
*   **Embedding Model**: These are generally much smaller than LLMs and require less RAM (e.g., `all-MiniLM-L6-v2` is ~90MB).
*   **Vector Database (ChromaDB)**: For in-memory ChromaDB, RAM usage scales with the number and size of your document chunks. For large datasets, consider a more robust, standalone vector database like Pinecone, Weaviate, Qdrant, or Postgres with `pgvector`.

**Note:** If you're starting, use CPU-only models (`ollama run llama2` will default to CPU if no GPU is available) and smaller models to validate functionality before investing in dedicated hardware.

### Performance Tuning

*   **LLM Quantization**: Use quantized versions of models (e.g., `llama2:7b-chat-q4_K_M`) from Ollama for faster inference and lower memory usage.
*   **`k` for Retriever**: Experiment with the `k` parameter in `retriever = vector_store.as_retriever(search_kwargs={"k": k})`. `k` is the number of most relevant document chunks to retrieve. Too few, and the LLM might lack context; too many, and you might exceed the LLM's context window or dilute the context with less relevant information.
*   **Chunk Size and Overlap**: Optimizing `chunk_size` and `chunk_overlap` during document ingestion is crucial. It depends heavily on the nature of your documents. Semantic chunking (where chunks are grouped by meaning) can also improve retrieval quality.
*   **Embedding Model Choice**: Different embedding models have varying performance and quality. `all-MiniLM-L6-v2` is a good general-purpose model, but larger ones might provide better semantic understanding at the cost of speed and memory.

### Scalability and Data Persistence

*   **ChromaDB Persistence**: As demonstrated, ChromaDB can persist data to disk (`persist_directory`). This means your embeddings aren't lost when the application restarts.
*   **Dockerization**: Containerize your application (Python script) and Ollama for easier deployment and management on a server or Kubernetes cluster.
*   **External Vector Stores**: For truly large document sets or multi-user scenarios, you'd move away from an in-process ChromaDB to a dedicated vector database service (self-hosted Qdrant, Weaviate, or managed services like Pinecone, ChromaDB Cloud). This allows your chatbot application to be stateless and scalable.

### Security and Access Control

*   **Ollama API Access**: By default, Ollama's API is accessible on `localhost`. If deploying on a server, configure your firewall to restrict access to the Ollama port (11434) to only your chatbot application.
*   **Sensitive Data**: Ensure the documents you're feeding into the system are appropriate for the intended users of the chatbot. Since you're self-hosting, the data stays within your control, but internal access policies still apply.
*   **Input/Output Sanitization**: If exposing a web interface, sanitize user inputs and LLM outputs to prevent injection attacks or unintended data exposure.

## Further Enhancements

This guide provides a solid foundation. Here are ideas for extending your self-hosted chatbot:

*   **More Document Loaders**: Integrate loaders for PDFs (`pypdf`), Word documents (`python-docx`), web pages (`UnstructuredURLLoader`), Notion, Confluence, etc. LangChain has a rich ecosystem of loaders.
*   **Web UI**: Build a simple web interface using frameworks like [Streamlit](https://streamlit.io/) or [Gradio](https://www.gradio.app/) to make the chatbot user-friendly for non-technical users.
*   **Chat History and Memory**: Implement conversational memory so the chatbot remembers previous turns in a conversation. LangChain offers various memory modules.
*   **Monitoring and Logging**: Set up logging for user queries, retrieved documents, and LLM responses to debug issues and understand usage patterns.
*   **Hybrid Search**: Combine vector similarity search with traditional keyword search (e.g., BM25) for more robust retrieval, especially for documents where exact keywords are important.
*   **Source Citation**: Improve the `return_source_documents` feature to present citations more clearly, linking directly to the source document or page if possible.

## Conclusion

You've now built a foundational self-hosted chatbot that can answer questions using your own private documentation. This setup gives you complete control over your data, your models, and your infrastructure, opening up a world of possibilities for internal knowledge management and automation.

The field of LLMs and RAG is evolving rapidly, but the core principles we've covered—embedding, vector search, and augmented generation—remain fundamental. Experiment with different models, refine your document processing, and continuously improve your bot's performance. The power of private, AI-powered knowledge is now at your fingertips.