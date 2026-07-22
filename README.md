# VectorDB Engine 

A high-performance vector database and RAG (Retrieval-Augmented Generation) system implemented entirely in C++ with a real-time web interface. Implements production-grade search algorithms (HNSW, KD-Tree, multi-threaded Brute Force) with binary persistence, API authentication, and a full document embedding pipeline powered by local LLMs.

---

## Key Features

- **4 Search Algorithms** — HNSW, KD-Tree, Brute Force, and Multi-threaded Brute Force with side-by-side benchmarking
- **3 Distance Metrics** — Cosine similarity, Euclidean, Manhattan
- **Real-time 2D PCA Visualization** — Interactive scatter plot showing semantic clusters in vector space
- **RAG Pipeline** — Document chunking, embedding via Ollama (nomic-embed-text, 768D), retrieval via HNSW, answer generation via llama3.2 with inline source citations
- **Binary Persistence** — Custom binary formats (VDBX/DDBX) for both vector and document databases; data survives server restarts
- **API Key Authentication** — Auto-generated keys, protected mutating endpoints, frontend auth indicator
- **Usage Analytics** — Per-endpoint call tracking, average latency, queries-per-minute, persisted to disk
- **Category Filtering** — Filter search results by semantic category at the index level
- **Single-binary Deployment** — Compiles to one executable, serves the full UI, no external dependencies at runtime

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        index.html                           │
│   PCA Scatter Plot · Search · Documents · RAG Chat · Stats  │
└──────────────────────────┬──────────────────────────────────┘
                           │ REST API (localhost:8080)
┌──────────────────────────▼──────────────────────────────────┐
│                       main.cpp                              │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐   │
│  │  HNSW    │  │ KD-Tree  │  │BruteForce│  │BF Parallel │   │
│  │ O(log N) │  │ O(log N) │  │  O(N·d)  │  │O(N·d / T)  │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └─────┬──────┘   │
│       └──────────────┴─────────────┴───────────────┘        │
│                      VectorDB (16D)                         │
│                      DocumentDB (768D)                      │
│                                                             │
│  ┌────────────┐  ┌─────────────┐  ┌───────────────────┐     │
│  │ Persistence│  │ Auth Layer  │  │  Usage Tracker    │     │
│  │ VDBX/DDBX  │  │ API Key     │  │  Per-endpoint     │      │ 
│  └────────────┘  └─────────────┘  └───────────────────┘      │
│                                                              │
│  ┌────────────────────────────────────────────────────┐      │
│  │            OllamaClient                            │      │
│  │   embed: nomic-embed-text    gen: llama3.2         │      │
│  └────────────────────────────────────────────────────┘      │
└──────────────────────────────────────────────────────────────┘
```

---

## Technical Highlights

### HNSW (Hierarchical Navigable Small World)
- Same algorithm used by Pinecone, Weaviate, and Chroma
- Multilayer navigable graph with exponential layer decay
- O(log N) approximate nearest neighbor search
- Parameters: M=16, M0=32, ef_construction=200

### Multi-threaded Brute Force
- Splits dataset across `hardware_concurrency()` threads
- Zero shared mutable state during computation (no locks on hot path)
- Thread-local partial results merged post-join
- Demonstrates parallel speedup in the benchmark UI

### Binary Persistence
- Custom compact binary formats with magic bytes and versioning
- VectorDB: `VDBX` format — id, metadata, category, embedding per record
- DocumentDB: `DDBX` format — id, title, text, high-dimensional embedding
- Auto-save on every write operation, load on startup

### RAG with Citations
- Text chunking with configurable overlap (250 words, 30-word overlap)
- Semantic retrieval via HNSW over 768D embeddings
- LLM prompted to produce inline `[1]`, `[2]`, `[3]` source citations
- Frontend renders citations as clickable badges linking to source chunks

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | C++17 |
| HTTP Server | cpp-httplib (header-only) |
| Embeddings | Ollama + nomic-embed-text |
| LLM | Ollama + llama3.2 |
| Frontend | Vanilla JS, Canvas API, CSS3 |
| Persistence | Custom binary format |
| Threading | std::thread (C++ standard library) |

---

## API Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/search` | No | K-NN search with algorithm/metric/category selection |
| POST | `/insert` | Yes | Insert a vector |
| DELETE | `/delete/:id` | Yes | Delete a vector |
| GET | `/items` | No | List all vectors |
| GET | `/benchmark` | No | Compare all 4 algorithms |
| POST | `/doc/insert` | Yes | Embed and store a document |
| POST | `/doc/ask` | No | Full RAG pipeline (retrieve + generate) |
| GET | `/doc/list` | No | List stored documents |
| DELETE | `/doc/delete/:id` | Yes | Delete a document chunk |
| GET | `/auth/validate` | — | Validate API key |
| GET | `/stats/usage` | No | Per-endpoint usage analytics |
| GET | `/hnsw-info` | No | HNSW graph topology |

---

## Quick Start

```bash
# Prerequisites: g++ (C++17), Ollama with nomic-embed-text + llama3.2

# Compile
g++ -std=c++17 -O2 main.cpp -o db -lws2_32    # Windows
g++ -std=c++17 -O2 main.cpp -o db -lpthread    # Linux/Mac

# Run
./db

# Output:
# === VectorDB Engine ===
# http://localhost:8080
# 20 vectors | 16 dims | HNSW+KD-Tree+BruteForce
# API Key: a3f8c1...
# Ollama: ONLINE

# Open browser → http://localhost:8080
```

---

## Screenshots

The web UI features:
- Interactive 2D PCA scatter plot with animated query visualization
- Algorithm speed comparison (4-bar benchmark)
- Document embedding with automatic chunking
- RAG chat with typewriter streaming and clickable citations
- Live usage analytics dashboard
- API key authentication with visual status indicator

---

## What I Built & Learned

- Implemented HNSW graph construction and search from the paper (no libraries)
- Built a KD-Tree with hyperplane pruning for exact NN search
- Designed a compact binary serialization format for database persistence
- Implemented lock-free parallel computation with thread-local accumulation
- Built a complete RAG pipeline: chunking → embedding → retrieval → generation
- Created a REST API with authentication middleware
- Developed real-time data visualization with Canvas 2D and PCA projection

---

## License

MIT
