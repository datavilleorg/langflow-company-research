# Langflow Company Research

A Langflow flow that builds a sourced company research report from Wikipedia and SEC EDGAR through Dataville's MCP server.

This repo is the runnable companion to the Dataville blog post and docs guide. Create a blank Langflow project, add the Dataville MCP server, drag in the flow, and run the prompt. You get a report with citations you can check.

## What you'll build

The example asks for a research report on Walmart. The agent:

- Pulls company context from Wikipedia.
- Pulls the latest 10-K and 10-Q from SEC EDGAR.
- Uses the 10-K response for financials, recent filings, and insider transactions.
- Writes a report with a source and date for each section.

Sample output: [`examples/walmart-report.md`](examples/walmart-report.md).

![Walmart company research report](screenshots/report.png)

## How it works

```text
Chat Input
    ↓
Agent  ←  MCP Tools (Dataville, Tool Mode)
    ↓
Chat Output
```

| Component | Role |
| --- | --- |
| Chat Input | Takes the company name or question. |
| Agent | Decides when to call Dataville and writes the report. |
| MCP Tools | Exposes Dataville's `search_dataville` and `list_dataville_sources` tools. It must be in Tool Mode. |
| Chat Output | Displays the report. |

![Langflow company research flow](screenshots/flow-canvas.png)

The agent never queries Wikipedia or SEC directly. It calls `search_dataville` with a source (`wikipedia` or `edgar`) and a query. Dataville returns a normalized record. The agent reads `body` for the content and uses `metadata` for citations, dates, and figures.

The 10-K response also includes `metadata.recent_filings` and `metadata.insider_transactions`, so the agent does not need a separate Form 4 search. It still runs a separate 10-Q search for quarterly financials.

## Prerequisites

- Langflow 1.12.0 (Desktop or Docker).
- A model provider key (OpenAI, Anthropic, or another provider Langflow supports).
- A Dataville API key for regular use. Anonymous access works at 50 requests per 24 hours per IP.

Langflow Desktop opens on `http://localhost:7868`. The Docker image serves the full app on `http://localhost:7860`.

## 1. Create a blank project and add the Dataville MCP server

Create a blank project in Langflow. Open the MCP sidebar, choose **Add MCP Server**, and use the **Streamable HTTP/SSE** tab:

| Field | Value |
| --- | --- |
| Name | `dataville` |
| URL | `https://api.dataville.com/mcp` |
| Headers | `Authorization: Bearer YOUR_DATAVILLE_API_KEY` (optional) |

Use lowercase `dataville` for the name. The flow looks for that exact server name.

Or paste this into the **JSON** tab:

```json
{
  "mcpServers": {
    "dataville": {
      "url": "https://api.dataville.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_DATAVILLE_API_KEY"
      }
    }
  }
}
```

Leave out `headers` to try anonymous access.

Add the server before dragging in the flow.

## 2. Drag and drop the flow

Drag [`flows/company-research.json`](flows/company-research.json) onto the blank canvas. This is the easiest way to load the flow; the Import button is not required. Langflow loads the flow and connects the MCP Tools component to the `dataville` server.

The flow contains a Chat Input, an Agent, an MCP Tools component connected to the Agent's Tools port, and a Chat Output. The Chat Input is preloaded with the Walmart prompt. The MCP Tools component must be in **Tool Mode**, or the agent will not see the Dataville tools.

## 3. Set the model and keys

Open the Agent and choose your model provider. The exported flow is set to `anthropic/claude-opus-4.7` through OpenRouter, so change it if you use another provider. Add the provider key in Langflow's model settings, not in the flow file.

If you use a Dataville key, put it in the MCP server headers from step 1. The flow does not store keys.

## 4. Run it

Open the Playground. The Walmart prompt is already in the Chat Input. Send it:

```text
Build a company research report on Walmart (WMT). Use Dataville to pull company context from Wikipedia, the latest 10-K and 10-Q from SEC EDGAR, and insider activity. Cite the source for each section.
```

The full agent prompt is in [`prompts/agent-instructions.md`](prompts/agent-instructions.md).

## What the response gives you

Every Dataville source returns the same top-level shape: `status`, `account_state`, `data`, and `usage`. Inside `data`, every source returns `id`, `title`, `body`, `source`, `last_updated`, `entities`, and `metadata`.

- **Provenance:** `source`, `id`, `metadata.accession_number`, `metadata.cik`, and `metadata.document_url` tell you where a figure came from.
- **Freshness:** `metadata.filing_date`, `metadata.report_date`, `last_updated`, and `metadata.recent_filings` tell you what the data covers and what has happened since.
- **Consistency:** the same shape works for Wikipedia and EDGAR, so the agent needs no per-source parsing.

See [`examples/edgar-10k-response.json`](examples/edgar-10k-response.json) for a trimmed 10-K response.

![Dataville tool call in the Langflow Playground](screenshots/tool-call.png)

## Troubleshooting

**The agent has no Dataville tools.** Open the MCP Tools component and enable **Tool Mode**. When it is on, the component shows a Toolset output and the tool parameter fields disappear. Connect Toolset to the Agent's Tools port.

**The flow loads but the MCP component is disconnected.** Add the server as lowercase `dataville` before dragging in the flow, or re-select the server on the MCP Tools component after loading.

**429 responses.** Anonymous access is limited to 50 requests per 24 hours per IP. Add a Dataville API key in the MCP server headers.

**The model key is missing.** Set the provider key in Langflow's model settings. The flow does not include keys.

**The flow looks different after loading.** Flow JSON can drift between Langflow releases. This flow was built with 1.12.0. Update Langflow or re-create the MCP Tools component if the ports do not match.

## Repo structure

```text
.
├── flows/
│   └── company-research.json
├── prompts/
│   └── agent-instructions.md
├── examples/
│   ├── walmart-report.md
│   └── edgar-10k-response.json
└── screenshots/
    ├── flow-canvas.png
    ├── tool-call.png
    └── report.png
```

## Related

- Blog post: coming soon
- Dataville: https://www.dataville.com
- Docs: https://docs.dataville.com
- Video walkthrough: coming soon
- Dataville MCP server: https://github.com/datavilleorg/dataville-mcp
- SEC EDGAR dataset: https://github.com/datavilleorg/dataville-sec-edgar
- Wikimedia dataset: https://github.com/datavilleorg/dataville-wikimedia
- Code version of this use case: https://github.com/datavilleorg/company-intelligence-agent

## Contributing

Issues and pull requests are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Security

Do not commit API keys or exported flows that contain keys. See [SECURITY.md](SECURITY.md).

## License

MIT. See [LICENSE](LICENSE).
