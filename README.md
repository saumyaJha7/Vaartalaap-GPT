# Vartalaap GPT

A small ChatGPT-style chat application built with Node.js, Express, and vanilla JavaScript. It uses the Groq API to run the `openai/gpt-oss-120b` model and can automatically call a **Tavily web search** tool when the model needs real-time or unknown information. Conversations are kept per browser session using an in-memory cache, and the UI is styled with Tailwind CSS.

## Features

- Chat interface modeled after ChatGPT
- Groq-hosted `openai/gpt-oss-120b` model
- Automatic web search via Tavily when the model decides it needs external information
- Per-thread conversation memory using `node-cache` (1 hour TTL)
- Loading indicator while the model responds
- Plain-text responses with light markdown/table cleanup on the server

## Tech Stack

| Layer | Technology |
| --- | --- |
| Backend | Node.js, Express 5 |
| LLM | Groq SDK (`openai/gpt-oss-120b`) |
| Web search | Tavily (`@tavily/core`) |
| Memory | `node-cache` |
| Frontend | HTML, Vanilla JS |
| Styling | Tailwind CSS 4 |
| Config | `dotenv` |

## Project Structure

```
.
├── Llm.js              # LLM logic, tool routing, web search, and thread memory
├── server.js           # Express server and /chat API
├── frontend/
│   ├── index.html      # Chat UI
│   └── script.js       # Client-side chat logic
├── src/
│   ├── input.css       # Tailwind entry point
│   └── output.css      # Generated Tailwind CSS
├── package.json
└── .env                # API keys (not committed)
```

## Getting Started

### Prerequisites

- Node.js 18+
- A [Groq API key](https://console.groq.com/keys)
- A [Tavily API key](https://app.tavily.com/)

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
```

### 3. Build/watch Tailwind CSS

```bash
npm run css
```

This watches `src/input.css` and regenerates `src/output.css`.

### 4. Start the server

```bash
npm start
```

The API runs at `http://localhost:3000`.

### 5. Open the frontend

Serve the `frontend` directory with any static server (for example, the VS Code **Live Server** extension) and open `index.html`. The client calls `http://localhost:3000/chat`, so the backend must be running.

## API

### `GET /`

Health-check-style response:

```text
Server se hello !
```

### `POST /chat`

Request:

```json
{
  "message": "What is the latest news in AI?",
  "threadId": "unique-thread-id"
}
```

Response:

```json
{
  "message": "The assistant's reply..."
}
```

`threadId` is used as the key for the in-memory conversation cache. Send the same ID across requests to continue a conversation.

## How It Works

1. The client sends the user message and a `threadId` to `POST /chat`.
2. `Llm.js` loads the conversation history for that thread (or starts with the system prompt).
3. A lightweight classification call asks the model whether an external tool is needed.
4. If not needed, the model answers directly.
5. If needed, the model is called with a `webSearch` tool definition. The server executes the Tavily search, appends the tool result to the conversation, and lets the model produce a final answer.
6. The updated conversation is cached for one hour under `threadId`.

## Known Limitations

- Conversation history is stored in memory and is lost when the server restarts; it is not shared across multiple server instances.
- The retry logic in `Llm.js` currently returns the fallback message on the first tool-loop iteration (`if (count < MAX_RETRIES)`) instead of retrying up to the limit.
- `frontend/script.js` uses `Math.random.toString(36)` (missing `()`), so `threadId` is not as random as intended.
- The send-button handler passes `"demo"` as the role instead of `"assistant"`, and some HTML attributes contain minor typos.
- There is no authentication, rate limiting, or input validation. Do not expose this server publicly as-is.

## Scripts

| Command | Description |
| --- | --- |
| `npm start` | Start the Express server |
| `npm run css` | Watch and rebuild Tailwind CSS |
| `npm test` | Placeholder (no tests configured) |
