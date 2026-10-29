# Contributing

Thanks for your interest. Issues and pull requests are welcome.

- **Bugs / ideas:** open an issue with what you ran, what you expected, and what happened. Include your Langflow version.
- **Pull requests:** keep them focused, and describe how you tested the change.
- **Provenance is a feature:** keep citations and source metadata intact in examples and prompts.
- **Secrets:** never commit an API key or an exported flow that contains one. Use `.env` (git-ignored); see `.env.example`.

## Flow changes

The flow is exported from Langflow, not hand-written.

1. Create a blank project and add the `dataville` MCP server. Use lowercase `dataville`.
2. Drag `flows/company-research.json` onto the canvas.
3. Export it back to `flows/company-research.json` from the flow menu.
4. Remove API keys, bearer headers, model provider keys, and global variable values. Check the MCP Tools headers and any `env` fields.
5. Test the drag-and-drop on a fresh Langflow instance with a newly added `dataville` server.
6. Confirm the MCP Tools component is in Tool Mode and the agent calls `search_dataville`.
7. Record the Langflow version in the README.

Flow JSON can drift between Langflow releases.

Questions about the Dataville API itself: support@dataville.com.
