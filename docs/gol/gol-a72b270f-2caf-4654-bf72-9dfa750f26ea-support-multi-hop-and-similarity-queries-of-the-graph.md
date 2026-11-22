---
classification: null
created: '2026-10-13T03:49:45.243+02:00'
id: a72b270f-2caf-4654-bf72-9dfa750f26ea
status: draft
type: gol
updated: '2026-10-13T03:49:45.243+02:00'
version: 1.0.0
---

# Support Multi-Hop and Similarity Queries of the Graph

GraphMgr answers graph traversals beyond direct-neighbor lookups (multi-hop) and semantic similarity search over the name and description of nodes and edges, against a query index derived from the text files.

## Description

Text files alone are weak at multi-hop queries, so the design loads them into an in-memory relational index (SQLite) with change detection, giving a strong query language (SQL, WITH RECURSIVE) plus a vector-similarity search over locally computed embeddings (no external API calls).

## Priority

20

## Tags

- query
- multi-hop
- similarity

## Source

Clarification answers in session ses_ed8c0da78ffeLB78W5hsXJNOor (multi-hop preferred; in-memory relational DB with change detection); vector search requirement recorded in first-conversation.md sections 2 and 5.7.

## More Information

Raises PRB e1dd6083-cbd6-42da-a285-bf9e80c21d5a: Generic Typed Node/Link Graph with Git-Trackable Text Storage.
