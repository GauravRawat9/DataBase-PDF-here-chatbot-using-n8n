# DataBase-PDF-here-chatbot-using-n8n

A production-ready RAG (Retrieval-Augmented Generation) chatbot that lets you chat with Machine Learning PDF document — built entirely with n8n, Pinecone, and OpenAI — no custom code required beyond a single chunking script.

🚀 Demo
Ask questions like:

"What is supervised learning?"
"Explain overfitting in simple terms"
"What is the difference between training and test data?"

The bot retrieves the most relevant chunks from the book and answers accurately — without hallucinating.

🧠 How It Works
This project uses the RAG (Retrieval-Augmented Generation) pattern:

1. The PDF is downloaded from Google Drive and text is extracted
2. Text is chunked into ~500-word segments
3. Each chunk is embedded using OpenAI's text-embedding-3-small model
4. Embeddings are stored in a Pinecone vector database (1536 dimensions)
5. At query time, the user's question is embedded and matched against stored vectors
6. The top 6 most relevant chunks are retrieved and passed to GPT-4o-mini
7. The LLM answers strictly based on retrieved context


🏗️ Architecture
Workflow 1 — Ingestion (run once)
```
Google Drive → Extract From PDF → Code (chunker) → Split Out → Edit Fields → Pinecone Vector Store (Insert)
                                                                               ↓
                                                                       Embeddings OpenAI
                                                                       (text-embedding-3-small)
```
Workflow 2 — Chat (runs on every message)
```
Chat Trigger → AI Agent
                  ├── OpenAI Chat Model (gpt-4o-mini)
                  ├── Simple Memory (Window Buffer)
                  └── Pinecone Vector Store (Retrieve as Tool)
                              ↓
                      Embeddings OpenAI
                      (text-embedding-3-small)
```
🛠️ Tech Stack
n8n --> Workflow automation & orchestration
Pinecone --> Vector database for semantic search
OpenAI --> Embeddings (text-embedding-3-small) + LLM (gpt-4o-mini)
Google Drive --> PDF source storage

⚙️ Setup & Installation
Prerequisites

- n8n instance (cloud or self-hosted)
- Pinecone account (free tier works)
- OpenAI API key
- Google Drive with your PDF

Step 1 — Pinecone Setup

1. Create a new index in Pinecone
2. Set dimensions to 1536
3. Set metric to cosine
4. Note your index name

Step 2 — Ingestion Workflow

1. Import or recreate the ingestion workflow in n8n
2. Configure Google Drive node with your file ID
3. Add the chunking Code node:
```bash
javascriptconst text = $input.first().json.text;
const chunkSize = 500;
const words = text.split(' ');
const chunks = [];

for (let i = 0; i < words.length; i += chunkSize) {
  chunks.push(words.slice(i, i + chunkSize).join(' '));
}

return [{ json: { chunks: chunks } }];
```
4. Configure Pinecone Vector Store → Insert Documents → your index name
5. Attach Embeddings OpenAI → text-embedding-3-small
6. Execute the workflow once — wait for all chunks to be indexed

Step 3 — Chat Workflow

1. Create a new workflow with Chat Trigger
2. Add AI Agent node with this system prompt:
```
You are a helpful assistant that answers questions strictly based 
on the book content provided to you. If the answer is not found 
in the book, say "I couldn't find that in the book." 
Do not make up answers.
```
3. Attach sub-nodes to AI Agent:

- OpenAI Chat Model → gpt-4o-mini
- Simple Memory → Window Buffer
- Pinecone Vector Store → Retrieve Documents (As Tool) → same index → text-embedding-3-small


4. Click Open Chat to test


📊 Performance

- Book size tested: 400 pages (~183 chunks at 500 words each)
- Indexing time: ~5–8 minutes (OpenAI rate limits on free tier)
- Query response time: ~2–4 seconds
- Vectors stored: ~1000+ (n8n splits chunks further internally)


💡 Key Design Decisions

- Chunk size of 500 words balances context richness vs. retrieval precision
- text-embedding-3-small chosen for cost efficiency — $0.02 per 1M tokens
- Limit of 6 chunks per query gives the LLM enough context without hitting token limits
- Window Buffer Memory maintains conversation history for follow-up questions


🔧 Customization

- Change chunkSize in the Code node to adjust granularity (try 300 for more precise retrieval)
- Swap gpt-4o-mini for gpt-4o for higher quality answers
- Add chunk overlap by modifying the Code node for better boundary handling
- Change the system prompt to tune the chatbot's personality

🙋 Author
Built with n8n's visual workflow builder — no backend code, no deployment complexity, fully extensible.
