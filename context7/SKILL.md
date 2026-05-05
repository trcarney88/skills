---
name: context7
description: |
  Use the Context7 MCP tools to fetch current library documentation. Activate when you need current
  docs for any library, or when the user asks to check docs, mentions "ctx7", or mentions "context7".
license: MIT
compatibility: opencode
metadata:
  audience: engineers
  tool: context7-mcp
---

# Context7 MCP

Context7 provides up-to-date library documentation through MCP tools. Prefer the MCP tools when they are available:

- `context7_resolve-library-id` - resolve a package/product name to a Context7-compatible library ID.
- `context7_query-docs` - retrieve and query current documentation for a resolved library ID.

Only use the `bunx ctx7` CLI as a fallback if the MCP tools are unavailable.

# Quick Reference

1. Resolve the library ID with `context7_resolve-library-id` unless the user already provided a Context7 library ID in `/org/project` or `/org/project/version` format.
2. Query documentation with `context7_query-docs` using the resolved library ID.
3. If the first documentation query is insufficient, retry `context7_query-docs` with `researchMode: true`.

# Documentation Commands

Retrieves and queries up-to-date documentation and code examples from Context7 for any programming library or framework. Two-step workflow: resolve the library name to get its ID, then query docs using that ID.

If the user already provided a library ID in `/org/project` or `/org/project/version` format, pass it directly to `context7_query-docs`.

## Step 1: Resolve a Library

Resolves a package/product name to a Context7-compatible library ID and returns matching libraries.

Use `context7_resolve-library-id` with:

- `libraryName`: the official package or product name, for example `React`, `Next.js`, or `Prisma`.
- `query`: the user's documentation question or task, written without sensitive or confidential details.

Always pass a `query` argument - it is required and directly affects result ranking. Use the user's intent to form the query, which helps disambiguate when multiple libraries share a similar name. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.

### Result fields

Each result includes:

- Library ID - Context7-compatible identifier (format: `/org/project`)
- Name - Library or package name
- Description - Short summary
- Code Snippets - Number of available code examples
- Source Reputation - Authority indicator (High, Medium, Low, or Unknown)
- Benchmark Score - Quality indicator (100 is the highest score)
- Versions - List of versions if available. Use one of those versions if the user provides a version in their query. The format is `/org/project/version`.

### Selection process

1. Analyze the query to understand what library/package the user is looking for
2. Select the most relevant match based on:
   - Name similarity to the query (exact matches prioritized)
   - Description relevance to the query's intent
   - Documentation coverage (prioritize libraries with higher Code Snippet counts)
   - Source reputation (consider libraries with High or Medium reputation more authoritative)
   - Benchmark score (higher is better, 100 is the maximum)
3. If multiple good matches exist, acknowledge this but proceed with the most relevant one
4. If no good matches exist, clearly state this and suggest query refinements
5. For ambiguous queries, request clarification before proceeding with a best-guess match

IMPORTANT: Do not call `context7_resolve-library-id` more than 3 times per question. If you cannot find what you need after 3 calls, use the best result you have.

### Version-specific IDs

If the user mentions a specific version, use a version-specific library ID:

Use the general ID for the latest indexed docs, for example `/vercel/next.js`.

Use a version-specific ID when the user asks about a specific version, for example `/vercel/next.js/v14.3.0-canary.87`.

The available versions are listed in the `context7_resolve-library-id` output. Use the closest match to what the user specified.

## Step 2: Query Documentation

Retrieves up-to-date documentation and code examples for the resolved library.

You must call `context7_resolve-library-id` first to obtain the exact Context7-compatible library ID required to use this command, UNLESS the user explicitly provides a library ID in the format `/org/project` or `/org/project/version`.

Use `context7_query-docs` with:

- `libraryId`: the exact Context7 library ID, for example `/reactjs/react.dev`, `/vercel/next.js`, or `/prisma/prisma`.
- `query`: the user's documentation question or task, written specifically and without sensitive or confidential details.
- `researchMode`: `false` for the first query; retry with `true` only if the first result does not answer the question.

IMPORTANT: Do not call `context7_query-docs` more than 3 times per question. If you cannot find what you need after 3 calls, use the best information you have.

### Writing good queries

The query directly affects the quality of results. Be specific and include relevant details. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.

| Quality | Example |
|---------|---------|
| Good | `"How to set up authentication with JWT in Express.js"` |
| Good | `"React useEffect cleanup function with async operations"` |
| Bad | `"auth"` |
| Bad | `"hooks"` |

Use the user's full question as the query when possible - vague one-word queries return generic results.

The output contains two types of content: code snippets (titled, with language-tagged blocks) and info snippets (prose explanations with breadcrumb context).

# Common Mistakes

- Library IDs require a `/` prefix - `/facebook/react` not `facebook/react`
- Always run `context7_resolve-library-id` first unless the user provides a valid `/org/project` or `/org/project/version` library ID.
- Do not use the CLI when MCP tools are connected and available.
- Do not include API keys, credentials, personal data, or proprietary code in Context7 queries.

# CLI Fallback

If MCP tools are unavailable, use the CLI exactly like this without installing anything:

```bash
bunx ctx7 library <name> <query>
bunx ctx7 docs <libraryId> <query>
```
