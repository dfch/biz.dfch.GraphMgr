---
classification: null
created: '2026-10-13T03:49:28.010+02:00'
id: 86b457d9-58b0-4c9c-8c53-cf7849a72e75
status: draft
type: gol
updated: '2026-10-13T03:49:28.010+02:00'
version: 1.0.0
---

# Provide a Generic, Extensible Typed Node/Edge Graph Model

GraphMgr provides a domain-agnostic graph in which nodes and edges carry a fixed core attribute set plus type-specific attributes, and new types can be introduced by adding a schema file without deploying new code.

## Description

The model must stay generic: it is not tied to one domain (architecture, organization, processes), and it must support UML-like typed, directed connectors that carry their own attributes (label, multiplicity, navigability, weight, ...). Edges are first-class stored entities, not inline references inside node files.

## Priority

10

## Tags

- data-model
- extensibility

## Source

User statement in session ses_ed8c0da78ffeLB78W5hsXJNOor (2026-10-10); recorded in first-conversation.md sections 1-2 and 3.1.

## More Information

Raises PRB e1dd6083-cbd6-42da-a285-bf9e80c21d5a: Generic Typed Node/Link Graph with Git-Trackable Text Storage.
