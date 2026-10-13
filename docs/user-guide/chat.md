# Chat Interface

The chat interface is the primary way to interact with EpidBot.

## Accessing the Chat

Navigate to `/chat` in your browser. If you're not logged in, you'll be redirected to the login page.

- **New chat**: Go to `/chat` or click "New Chat" in the sidebar
- **Continue session**: Click on a previous session in the sidebar, or navigate to `/chat/{sessionId}`

## Interface Elements

### Chat Window

Displays the conversation history with:

- **User messages** (right-aligned)
- **Assistant responses** (left-aligned, with markdown rendering)
- **Images** (plots, maps, and visualizations displayed inline)
- **Tool progress** (real-time indicators when tools are running)

### Tool Progress Indicators

When the agent executes tools, you'll see:

- Tool name and elapsed time
- Code preview for sandbox/SQL executions
- Completion status (success, error, or timeout)

### Input Box

Type your question and press **Enter** or click **Send**.

!!! tip "Mid-stream Messaging"
    You can send a new message while the assistant is still streaming. The current response will be interrupted and saved, and a new turn will begin with your new message.

### Voice Input

Click the **microphone button** inside the input field to use voice input. Speak naturally and your words will be transcribed in real time. Click the microphone again to stop recording. The recognized language follows your interface locale (configurable under *Settings → Voice*).

On browsers without built-in speech recognition (e.g., Firefox), the microphone automatically falls back to server-side transcription: your audio is sent to the EpidBot server, transcribed, and inserted into the input box. This requires the server's voice feature to be enabled; on a spinning state, the transcription is in flight.

### Read Aloud & Auto-read

Each assistant message has a **read aloud** button in its action bar. Enable **Auto-read** in the chat toolbar (or *Settings → Voice*) to have responses spoken as they stream in, sentence by sentence. Code blocks, tables, and formulas are skipped — only the prose is read. Reading speed and voice can be configured under *Settings → Voice*; when the server offers natural (synthesized) voices, you can choose them there too.

### Voice Mode

Open hands-free **Voice mode** from the chat toolbar or with **Ctrl+Shift+V** (Esc closes it). Voice mode loops continuously: it listens while you speak, a natural pause sends your message, spoken status updates play while the assistant works ("Let me check the data."), and the response is read aloud as it streams. While the assistant speaks, tap anywhere to interrupt and speak again — the assistant handles mid-stream interruptions natively. If the assistant asks a clarifying question, it is read aloud and your spoken answer is sent as the reply.

### Session Sidebar

The session sidebar on the left side of the chat can be **collapsed** by clicking the collapse button in its header, giving you more space for the conversation. Click the expand button to bring it back.

### Suggestion Cards

When you start a new chat, the welcome screen displays **analysis starter templates** — clickable suggestion cards with common analysis prompts such as "Analyze dengue trends" or "Compare health indicators." Click any card to immediately start an analysis.

## Onboarding Tutorial

New users automatically receive a guided tutorial on their first visit. The tutorial introduces:

1. Available datasets and data sources
2. How to download and query data
3. International health data analysis
4. Visualization capabilities
5. Tips for complex analyses

The tutorial triggers automatically when a new user with no previous sessions opens the chat.

## Editing Messages

You can edit your **last sent message** by hovering over it and clicking the **pencil icon**. Edit the text in the inline editor and press **Enter** to resubmit. EpidBot will discard the previous response and generate a new one based on your edited message. Press **Escape** to cancel.

## Message Feedback

After EpidBot responds, hover over an assistant message to reveal **thumbs up** and **thumbs down** buttons. Click to rate the quality of the response. Click the same rating again to remove it. This feedback helps improve the system over time.

## Exporting Chat

Click the **Export PDF** button at the top of the chat to download the entire conversation as a formatted PDF document, including all messages, embedded images, and formatting.

You can also export sessions from the Sessions page in multiple formats: **Markdown, JSON, HTML, plain text, and PDF**.

## Rich Content Features

### Code Blocks

Code blocks are syntax-highlighted with a dark theme. Hover over a code block to reveal a **copy button** that copies the code to your clipboard.

### Data Tables

Tables rendered in responses include hover actions:

- **Copy as TSV** — copies table data to clipboard in tab-separated format
- **Download CSV** — downloads table data as a CSV file

### Markdown Rendering

Responses support:

- **Bold** and *italic* text
- Lists (numbered and bulleted)
- Code blocks with syntax highlighting
- Tables with copy and download actions
- Mathematical notation (KaTeX)
- Inline images (plots, maps, diagrams)

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Enter` | Send message |
| `Shift+Enter` | New line |
| `Ctrl+K` | Open search |
| `Ctrl+N` | Start new session |
| `Ctrl+Shift+E` | Export chat as PDF |
| `Ctrl+Shift+V` | Toggle voice mode |
| `Escape` | Close lightbox / cancel edit / exit voice mode |

## Interactive Cards

Besides plots and tables, the chat can render interactive cards:

- **Clarifying questions** — when the agent needs a decision from you, it shows clickable option cards (also spoken in voice mode)
- **Continue analysis** — suggested next steps you can trigger with one click
- **Task progress** — a live checklist for long, multi-step turns
- **Suggestion cards** — starter prompts on the empty chat page

## Example Queries

### Download Data

```
Baixe os dados de Dengue para 2023
```

```
Download Zika data for 2022 and 2023
```

### Query Data

```
Quantos casos de dengue foram reportados em São Paulo em 2023?
```

```
Show the distribution of dengue cases by age group
```

### Generate Charts

```
Gere um gráfico de linha mostrando a tendência temporal
```

```
Create a bar chart of cases by state
```

### Maps and Raster Data

```
Mostre um mapa dos casos de dengue por município
```

```
Download the malaria prevalence raster and plot it
```

### International Data

```
What are the latest WHO disease outbreaks?
```

```
Download World Bank life expectancy data for South American countries
```

## Connection Status

A status indicator appears below the input when the WebSocket connection is not active:

- **Reconnecting...**: Connection lost, automatically retrying with exponential backoff
- The send button is always available — messages are queued when offline and sent on reconnection

## Best Practices

1. **Start with data discovery**: Check what's available before complex queries
2. **Be specific about years**: Always specify the year(s) of interest
3. **Request visualizations**: The agent works best when asked to create charts
4. **Use sessions**: Create separate sessions for different analyses
5. **Ask in any language**: English, Portuguese, and Spanish are well-supported
6. **Edit to refine**: Edit your last message to adjust parameters without retyping
7. **Consult external agents**: If you have registered an external agent (e.g. a nanobot instance) in **Settings > External Agents**, you can ask EpiDBot to contact it — e.g. *"Consult my agent named my-nanobot and ask: what models are available for dengue forecasting?"* EpiDBot sends the message to the agent's OpenAI-compatible API and relays the response back into the conversation.

## Error Handling

If an error occurs:

- A user-friendly message is displayed in the chat
- Common issues include quota limits, sandbox timeouts, or data not found
- You can retry by sending the message again
- Unanswered messages show an amber banner with a **Retry** button

Example error:
```
The AI model's API quota has been exceeded.
Please try again later or contact the administrator.
```
