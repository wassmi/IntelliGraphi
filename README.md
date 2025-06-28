# IntelliGraphi

Project: IntelliGraphi - AI-Powered Knowledge Graph Visualization & RAG Platform

[IntelliGraphi Live Demo](https://intelligraphi-180686635896.us-west1.run.app/)

Project Overview

IntelliGraphi is a sophisticated, browser-based application designed to bridge the gap between unstructured data and structured, actionable insights. At its core, it leverages a powerful Retrieval-Augmented Generation (RAG) pipeline to parse large documents (PDFs, TXT files) and transform their contents into interactive, high-performance knowledge graphs.

The platform allows users to upload complex documents and use a conversational AI to perform exhaustive analysis, generating detailed graphs of entities and their relationships. This automates the discovery of hidden connections and provides a dynamic, visual medium for data exploration, a task that is traditionally manual, slow, and error-prone.

# Core Features
AI-Driven Graph Generation: A conversational chat interface allows users to construct, modify, and query graphs using natural language.
Retrieval-Augmented Generation (RAG): Users can upload documents (PDF, TXT, CSV), which are processed through an intelligent pipeline (chunking, embedding, indexing) to provide factual, in-document context for all AI-powered analysis.

High-Performance Interactive Canvas: A fully custom-built graph editor supports fluid panning, zooming, multi-select (marquee, lasso), node pinning, and contextual interactions, ensuring a smooth experience even with large graphs.

Intelligent Layout Engine: Features multiple layout algorithms, including a robust "anti-hairball" force-directed layout and a structured radial layout, powered by a D3 simulation running in a dedicated Web Worker.

Full State Management: The application state, including graph structure, checkpoints, and chat history, is managed predictably and persisted in localStorage. Graphs can be exported to and imported from the standard GML format.

Advanced UI/UX: A multi-panel, resizable interface includes a live data view (GML), version history with one-click rollbacks, and a navigation minimap for large graphs.

# Technical Architecture Deep Dive
The application is built with a modern, performance-first architecture designed to handle complex computations and rendering directly in the browser.

1. Frontend & State Management:
Framework: Built with React 19 and TypeScript, ensuring a strongly-typed, component-based structure.
State Management: Utilizes React's useReducer hook for centralized and predictable state transitions. A custom usePersistentReducer hook serializes the entire application state to localStorage, providing session persistence. This hook includes a custom JSON replacer/reviver to correctly handle the serialization of Map and Set data structures, which are used extensively for performant data access.

2. Graph Visualization & Interaction (The Canvas):
Rendering Engine: The graph is rendered directly on the HTML5 Canvas API for maximum performance and control over every pixel. This avoids the overhead of DOM-based solutions like SVG for large-scale visualizations.
Custom Renderer Hook (useGraphRenderer): This hook encapsulates all drawing logic. It employs several optimization techniques:
Viewport Culling: Only nodes and edges within the current view are rendered.
Level of Detail (LOD): Node labels and other details are progressively hidden as the user zooms out, reducing visual clutter and draw calls.
Dynamic Styling: Visual properties (colors, stroke widths) are calculated on-the-fly to reflect state (selection, highlight, hover, analysis results).
Edge Bundling: Multiple edges between two nodes are drawn as elegant parallel curves, improving readability in dense graphs.
Interaction Model: All user interactions (panning, dragging, selection) are handled by calculating coordinate transformations between screen space and world space, enabling a fluid and intuitive navigation experience.

3. Physics Simulation & Layout Engine:
Decoupled Computation: The entire graph physics simulation is offloaded to a dedicated Web Worker (simulation.worker.ts). This is the key architectural decision that guarantees a completely smooth, non-blocking UI thread, even when the simulation for a large graph is actively running.
Physics Engine: Leverages the battle-tested d3-force library to power the simulation.
Intelligent Layouts: Implements a "clean slate" principle where changing the layout (e.g., from 'force-directed' to 'radial') completely destroys and rebuilds the simulation with a new set of fine-tuned forces. This ensures predictable and dramatic layout changes. The force-directed algorithm is configured with aggressive, context-aware repulsion to actively prevent "hairball" graphs.

4. AI & Retrieval-Augmented Generation (RAG) Pipeline:
The entire AI interaction for document analysis is built on a custom, in-browser RAG pipeline.
1. Ingestion & Parsing: Leverages pdfjs-dist to extract text content from PDF files.
2. Chunking: A custom RecursiveCharacterTextSplitter intelligently breaks down large texts into smaller, overlapping chunks, preserving semantic context.
3. Embedding: Utilizes the Google Gemini API (text-embedding-004 model) to convert each text chunk into a 768-dimension vector embedding. This process is:
Optimized: Uses the retrieval_document task type for better search performance.
Robust: API calls are batched using Promise.allSettled, which gracefully handles individual chunk failures (e.g., due to content filters) by using a zero-vector fallback, preventing the entire process from crashing.

4. Indexing & Search: A lightweight, in-memory VectorStore class stores the embeddings and performs fast similarity searches using cosine similarity.

5. Retrieval & Generation: When a user sends a query, the RAG pipeline retrieves the most relevant text chunks from the vector store. This context is then "augmented" into the system prompt for the main LLM (Gemini 2.5 Flash), instructing it to base its response and graph generation only on the provided facts. The AI is mandated to respond with structured JSON, which is then parsed and dispatched as actions to modify the application state.

# Key Technical Challenges & Solutions

Challenge 1: UI Unresponsiveness & "Hairball" Graphs.
Solution: Decoupled the D3 force simulation into a Web Worker, freeing the main thread for a fluid UI. Designed a more aggressive force-directed layout with repulsion that scales with graph density to actively push nodes apart.

Challenge 2: Processing Large Documents in the Browser.
Solution: Implemented the RAG pipeline. Instead of loading an entire file into the LLM's context, the document is chunked and embedded. Only the most relevant, contextually-aware chunks are used for any given query, making the system scalable and efficient.

Challenge 3: Performant Rendering for Hundreds of Nodes/Edges.

Solution: Built a custom renderer on the HTML5 Canvas from the ground up, implementing performance optimizations like viewport culling and Level of Detail (LOD) for labels to ensure a high frame rate.

# Technology Stack
Frontend: React, TypeScript, Tailwind CSS
AI/LLM: Google GenAI SDK (Gemini 2.5 Flash, Text Embedding 004)
Graph Engine: D3-Force
Core APIs: HTML5 Canvas API, Web Workers, File API
File Parsing: pdfjs-dist
