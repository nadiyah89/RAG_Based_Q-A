# 🏦 AI Loan Knowledge Assistant (RAG)

A fully local, privacy-first **Retrieval-Augmented Generation (RAG)** API built with **.NET 9**. This application allows users to upload PDF documents (like loan policies, FAQs, or CVs) and ask natural language questions about them. The system retrieves relevant context from the documents and generates accurate, grounded answers using local AI models.

Since it runs completely locally using **Ollama** and **Qdrant**, no data is ever sent to cloud providers (like OpenAI or Azure).

---

##  Features
*  PDF Ingestion: Upload any PDF document to automatically extract, chunk, and index its text.
*  100% Local AI: Powered by Ollama for both chat generation and vector embeddings.
*  Semantic Search: Uses Qdrant Vector Database for lightning-fast similarity search.
*  Clean Architecture: Highly decoupled layers (API, Application, Infrastructure, Domain) for easy scaling and swapping of technologies.
*  JWT Authentication: Built-in JWT bearer token support (configurable).

---

##  Tech Stack
* **Framework:** C# / .NET 9 Web API
* **Architecture:** Clean Architecture (Onion Architecture)
* **LLM (Generation):** `llama3.2` (via Ollama)
* **Embeddings:** `all-minilm` (via Ollama) - 384 dimensions
* **Vector Database:** Qdrant (via Docker)
* **PDF Processing:** PdfPig

---

##  Getting Started

### 1. Prerequisites
You will need the following installed on your machine:
* [.NET 9 SDK](https://dotnet.microsoft.com/download)
* [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Must be running)
* [Ollama](https://ollama.com/)

### 2. Setup Ollama (Local AI)
Once Ollama is installed, open a terminal and pull the required models:
```bash
ollama pull llama3.2
ollama pull all-minilm
```

### 3. Setup Qdrant (Vector Database)
Run Qdrant locally using Docker:
```bash
docker run -d -p 6333:6333 --name qdrant-loan qdrant/qdrant
```

Next, create the `loan_docs` collection with a vector size of **384** (which matches the output of the `all-minilm` model). Run this command in PowerShell:
```powershell
Invoke-RestMethod -Uri "http://localhost:6333/collections/loan_docs" `
  -Method Put `
  -Body '{"vectors":{"size":384,"distance":"Cosine"}}' `
  -ContentType "application/json"
```

### 4. Configuration
Ensure your `appsettings.json` in the `LoanAI.Api` project looks like this:
```json
{
  "Ollama": {
    "BaseUrl": "http://localhost:11434"
  },
  "Qdrant": {
    "BaseUrl": "http://localhost:6333"
  }
}
```

### 5. Run the Application
Navigate to the API project folder and run the app:
```bash
cd LoanAI.Api
dotnet run
```
Open your browser and navigate to the Swagger UI: `http://localhost:5163/swagger`

---

##  API Usage Flow

### Step 1: Ingest Documents
**`POST /api/documents/upload`**
* **Action:** Upload a PDF file.
* **What happens:** The text is extracted, split into overlapping chunks, embedded using `all-minilm`, and saved to Qdrant.

### Step 2: Ask a Question
**`POST /api/Loan/ask-rag`**
* **Payload:** `{"question": "What is the interest rate for a home loan?"}`
* **What happens:** The question is embedded, Qdrant retrieves the top 3 most relevant chunks from your PDF, and `llama3.2` reads those chunks to generate a factual answer.

### Other Endpoints
* `POST /api/Loan/ask` - Direct Q&A with the LLM (bypasses documents).
* `POST /api/Loan/embed` - Generate vector embeddings for a given text.
* `POST /api/Loan/store` - Manually store a specific chunk of text into Qdrant.
* `POST /api/Loan/search` - Perform a raw semantic search against the database.

---

##  Project Structure
```text
├── LoanAI.Api              # Entry point, Controllers, Swagger, Configuration
├── LoanAI.Application      # Interfaces and core abstractions
├── LoanAI.Domain           # Domain models and entities
└── LoanAI.Infrastructure   # Implementations (Ollama calls, Qdrant, PdfPig)
```

##  Contributing
Feel free to open issues or submit pull requests. Ensure all tests pass before submitting.
