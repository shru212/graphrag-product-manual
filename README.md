# GraphRAG Knowledge Graph

A self-contained GraphRAG demonstration built on n8n Cloud. The solution stores a typed knowledge graph in n8n Data Tables and compares relationship-aware GraphRAG answers with traditional Vector RAG answers on the same questions.

## Product manual

The complete configuration, operation, maintenance, troubleshooting, and testing guide is available here:

[Download the GraphRAG Product Manual v1.0](./GraphRAG_Product_Manual_v1.0.docx)

## What this project demonstrates

Traditional Vector RAG retrieves semantically similar entities independently. This works well for relevance-based lookup but often loses the relationships between the retrieved entities.

GraphRAG starts from a relevant entity and traverses typed relationships such as:

- `worked_on`
- `has_skill`
- `used_tool`
- `produced`
- `about`
- `made_decision`
- `approved`

This enables the system to answer relational questions such as:

- Who worked on the compliance audit, and what tools did they use?
- Which people worked on more than one project?
- What decisions were made about pricing, and who approved them?
- Which projects have no recorded decision?
- If a team member left, which projects and decisions would be affected?

## Architecture

The solution uses two n8n Data Tables as its graph store:

### `kg_nodes`

Stores graph entities, including people, projects, tools, documents, and decisions.

| Field | Purpose |
|---|---|
| `node_key` | Unique entity key |
| `node_type` | Entity category |
| `name` | Human-readable name |
| `description` | Entity description and folded properties |
| `embedding` | OpenAI embedding used for Vector RAG |

### `kg_edges`

Stores directed, typed relationships between entities.

| Field | Purpose |
|---|---|
| `from_key` | Source entity key |
| `to_key` | Target entity key |
| `relation` | Relationship type |
| `note` | Optional relationship context |

## Workflows

### Build Vector Index

The `GraphRAG Demo - Build Vector Index` workflow:

1. Reads all records from `kg_nodes`.
2. Generates an OpenAI embedding from each node description.
3. Writes the resulting vector to the node's `embedding` field.

Run this workflow after loading the graph and whenever node descriptions change.

### Knowledge Graph Chat

The `GraphRAG vs Vector RAG - Knowledge Graph Chat` workflow answers each question in two ways:

1. **GraphRAG:** identifies an entry node and traverses multiple typed edges to gather connected entities and relationships.
2. **Vector RAG:** ranks independently embedded nodes using cosine similarity.

The workflow returns both answers for side-by-side evaluation.

## Requirements

- An n8n instance; the documented solution uses n8n Cloud.
- An OpenAI credential connected in n8n.
- OpenAI account credits for embedding and answer-generation requests.
- Graph data expressed as supported Cypher `CREATE` or `MERGE` declarations.

## Getting started

1. Create the `kg_nodes` and `kg_edges` Data Tables using the schemas described above.
2. Load the node and relationship definitions into the tables.
3. Run `GraphRAG Demo - Build Vector Index`.
4. Confirm that every node has an embedding.
5. Open `GraphRAG vs Vector RAG - Knowledge Graph Chat`.
6. Submit a relational question and compare the two answers.

After any node-description update, rerun the vector-index workflow so that the embeddings remain synchronized.

## Example data

The demonstration dataset contains:

- 20 nodes
- 26 relationships
- 6 people
- 4 projects
- 5 skills or tools
- 3 documents
- 2 decisions

Example entities include Compliance Audit 2025, Pricing Model Revamp, Splunk, Vault, Adopt Usage-Based Pricing, and Migrate to AWS.

## Test findings

Across the documented test questions, GraphRAG consistently produced more complete relationship-aware answers. Vector RAG frequently retrieved the relevant entities but could not reliably explain how they were connected.

Examples include:

- GraphRAG connected Alice Rao and Fatima Aziz to Compliance Audit 2025 and identified Splunk and Vault as the project's tools.
- GraphRAG identified people connected to multiple projects by traversing `worked_on` relationships.
- GraphRAG connected decisions to their proposers and approvers.
- GraphRAG supported project and decision impact analysis for the departure of a team member.
- Vector RAG generally returned related names or concepts without the edges required to answer the relational question.

The system also preserved uncertainty. It did not invent an unrecorded rationale for the AWS decision or infer a person's skill without a supporting relationship.

## Known limitations

- The current graph is stored in n8n Data Tables rather than a native graph database.
- Graph traversal is implemented in an n8n Code node instead of Cypher.
- The loader accepts graph-definition statements but does not execute arbitrary Cypher read queries.
- Referential integrity is maintained manually.
- Embeddings must be regenerated after node-description changes.
- Indexing is not incremental.
- Chat turns do not retain conversational memory.
- The implementation is designed for small or medium knowledge graphs.
- Both embeddings and generated answers depend on a funded OpenAI credential.

For larger or production-grade graphs, consider moving the graph store and traversal layer to Neo4j while retaining n8n for orchestration.

## Security

Before publishing this repository:

- Remove or replace internal workflow, Data Table, and credential identifiers if they should not be public.
- Do not commit API keys, passwords, tokens, Neo4j credentials, or OpenAI credentials.
- Review personal names and organizational data for publication approval.
- Use a private GitHub repository when the graph or documentation contains internal information.

## Version

- Product manual: **1.0**
- Last updated: **2026-09-03**

## License

No license has been specified. Add a license file before allowing external reuse or redistribution.
