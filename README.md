# PageIndex RAG

Vectorless, reasoning-based RAG for PDFs. Instead of embedding chunks into a vector database, PageIndex builds a hierarchical tree from the document structure and uses LLM reasoning to navigate it — achieving 98.7% accuracy on FinanceBench.

> TypeScript port of [VectifyAI/PageIndex](https://github.com/VectifyAI/PageIndex) (MIT License). Original authors: Mingtian Zhang, Yu Tang, and the PageIndex Team at Vectify AI.

## How It Works

1. **Parse** — Extract text from each page of a PDF
2. **Detect TOC** — Scan the first N pages for a table of contents
3. **Build Tree** — Construct a hierarchical document tree (from TOC or generated from text)
4. **Verify** — Sample nodes and verify page mappings with the LLM
5. **Enrich** — Generate summaries for each node
6. **Search** — Present the tree structure to the LLM, which reasons about which nodes contain the answer

No vectors. No embeddings. No similarity search. Just structured reasoning.

## Installation

```bash
npm install pageindex-rag
```

## Quick Start

```typescript
import { setClient, buildPageIndex, searchAndAnswer } from 'pageindex-rag';

// Configure the OpenAI client
setClient('sk-your-openai-api-key');
// Or pass an existing OpenAI instance:
// setClient(new OpenAI({ apiKey: '...' }));

// Index a PDF
const tree = await buildPageIndex('./annual-report.pdf', {
  model: 'gpt-4o',
  addNodeSummary: true,
}, (status) => {
  console.log(`[${status.phase}] ${status.progress}% — ${status.message}`);
});

// Search the indexed document
const { answer, search } = await searchAndAnswer(
  'What was the total revenue in Q3?',
  tree,
  'gpt-4o',
);

console.log(answer);
console.log(`Found in nodes: ${search.node_ids.join(', ')}`);
console.log(`Context tokens: ${search.content_tokens}`);
```

## API Reference

### Configuration

```typescript
setClient(clientOrKey: OpenAI | string): void
```

Configure the OpenAI client. Must be called before indexing or searching.

### Indexing

```typescript
buildPageIndex(pdfPath: string, config?: Partial<PageIndexConfig>, onStatus?: StatusCallback): Promise<PageIndexTree>
```

Index a PDF file into a hierarchical tree structure.

**Config options:**

| Option | Default | Description |
|--------|---------|-------------|
| `model` | `'gpt-4o'` | OpenAI model for LLM calls |
| `tocCheckPageNum` | `20` | Pages to scan for TOC |
| `maxPagesPerNode` | `10` | Max pages before splitting |
| `maxTokensPerNode` | `20000` | Max tokens before splitting |
| `addNodeSummary` | `true` | Generate node summaries |
| `addDocDescription` | `true` | Generate document description |
| `addNodeText` | `false` | Include raw text in output |

### Searching

```typescript
searchTree(query: string, tree: PageIndexTree, model: string, maxContextTokens?: number): Promise<SearchResult>
```

Search an indexed document. Returns matched nodes' text.

```typescript
searchAndAnswer(query: string, tree: PageIndexTree, model: string, maxContextTokens?: number): Promise<{ answer: string; search: SearchResult }>
```

Search and generate an answer in one step.

## How It Compares to Vector RAG

| Aspect | Vector RAG | PageIndex |
|--------|-----------|-----------|
| Accuracy (FinanceBench) | ~60-70% | **98.7%** |
| Requires embeddings | Yes | No |
| Requires vector DB | Yes | No |
| Understands structure | No | Yes |
| Works with any LLM | Model-specific | Yes |
| Index portability | Embedding-locked | Plain JSON |

## Origin

Extracted from [Agent Friday](https://github.com/FutureSpeakAI/Agent-Friday), an AI desktop companion. Decoupled from Electron and made framework-agnostic for general use.

Based on the research and code from [VectifyAI/PageIndex](https://github.com/VectifyAI/PageIndex).

## License

MIT
