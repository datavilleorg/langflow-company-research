# Security policy

## Reporting a vulnerability

Please report vulnerabilities privately through GitHub's
[security advisory form](https://github.com/datavilleorg/langflow-company-research/security/advisories/new)
rather than a public issue. We aim to acknowledge reports within three business
days.

## Credentials

This repo does not include credentials. The Dataville API key is added to the
MCP server headers in Langflow, not to the flow file.

If you share an exported flow, check the MCP Tools component and global
variables for secrets first. Do not commit `.env` files or flow exports that
contain keys.

## Supported versions

This is an example repo. Fixes are applied to the `main` branch.
