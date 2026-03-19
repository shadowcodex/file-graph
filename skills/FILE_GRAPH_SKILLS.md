# File Graph Skills for Claude Code

Skill ideas for working with file graphs. These follow the [Agent Skills open standard](https://agentskills.io), so they'd work across Claude Code, Cursor, VS Code Copilot, Gemini CLI, Codex, Roo Code, Goose, and 30+ other tools.

---

## Core CRUD

### `/fg-init`
Initialize a new file graph. Creates the root directory structure, a starter `README.md` with the protocol spec, and optionally scaffolds initial node type directories based on user input.

- `disable-model-invocation: true` (side effects — creates files)
- Arguments: `$1` = root directory path, optional `$2` = template (e.g., `project`, `wiki`, `codebase`)

### `/fg-add`
Create a new node. Generates a `.md` file with the proper frontmatter template at the specified path. Prompts for name, description, and optional initial edges.

- Arguments: `$1` = node path (e.g., `jira/issues/PROJ-789.md`)
- Should validate the path fits an existing type directory or confirm creating a new type

### `/fg-link`
Create an explicit edge between two nodes. Updates the `explicit_edges` frontmatter in the source node. Optionally creates a bidirectional edge (updates both nodes).

- Arguments: `$1` = source node path, `$2` = target node path, `$3` = edge description
- `disable-model-invocation: true`

### `/fg-remove`
Remove a node and clean up. Deletes the file and scans the graph for any explicit edges or inline links pointing to it, offering to remove or update them.

- `disable-model-invocation: true` (destructive)
- Should show a preview of what will be affected before executing

### `/fg-move`
Move or rename a node. Updates the file path and rewrites all explicit edges and inline wikilinks/markdown links across the entire graph that reference the old path.

- Arguments: `$1` = old path, `$2` = new path
- `disable-model-invocation: true`

---

## Discovery & Navigation

### `/fg-find`
Search the graph. Query by tags, node type (directory pattern), edge relationships, content keywords, or any combination. Returns matching nodes with their descriptions.

- Arguments: freeform query string (e.g., `#bug product/ui`, `edges-to:people/engineers/alice.md`, `"validation error"`)

### `/fg-neighbors`
Show all nodes connected to a given node — both explicit edges and inline links, inbound and outbound. Supports depth parameter for n-hop traversal.

- Arguments: `$1` = node path, optional `$2` = depth (default: 1)
- Great for understanding the local context around any node

### `/fg-path`
Find the shortest path between two nodes by traversing edges. Useful for understanding how distant parts of the graph relate.

- Arguments: `$1` = start node, `$2` = end node

### `/fg-context` (Background Skill)
A non-user-invocable skill (`user-invocable: false`) that teaches the agent harness how to interpret and navigate the file graph format. Auto-loads when the agent encounters `.md` files with file-graph frontmatter patterns.

- This is arguably the **most important skill** — it's the one that makes every other interaction with the graph smarter
- Contains the protocol spec, conventions, and tips for efficient graph traversal
- Description should be tuned to trigger on file-graph related queries

---

## Analysis & Maintenance

### `/fg-validate`
Check graph integrity. Finds:
- Broken edges (pointing to nodes that don't exist)
- Orphaned nodes (no inbound edges from anywhere)
- Missing required frontmatter fields (`name`, `description`)
- Malformed wikilinks or markdown links
- Empty nodes (frontmatter only, no content)

### `/fg-stats`
Report on graph structure — node count by type, total edge count, most-connected nodes, tag distribution, orphan count. Uses dynamic context injection (`` !`command` ``) to pre-compute metrics.

### `/fg-index`
Build or rebuild index files at directory roots. Generates:
- Tag index (which nodes have which tags)
- Edge summary (inbound/outbound counts per node)
- Node listing with descriptions
- Writes these as markdown files at each type directory root

### `/fg-visualize`
Generate an interactive HTML visualization of the graph (or a subgraph). Nodes are circles, edges are lines, click to navigate. Uses the bundled-script pattern — outputs a single self-contained `.html` file.

- Arguments: optional `$1` = subtree to visualize (default: entire graph)
- Could also support Mermaid diagram output for embedding in docs

---

## Generation & Import

### `/fg-ingest`
Ingest external data into the graph. Takes a data source (CSV file, JSON file, API output) and a target node type path, creates properly formatted nodes with frontmatter.

- Arguments: `$1` = source file/URL, `$2` = target type path
- Should preview the first few nodes before creating all of them
- `disable-model-invocation: true`

### `/fg-from-codebase`
Bootstrap a file graph from an existing codebase. Analyzes imports, dependencies, exports, and file structure to auto-generate nodes and edges representing the code's architecture.

- Arguments: `$1` = codebase root path, optional `$2` = graph output path
- Language-aware — should handle JS/TS imports, Python imports, Go packages, etc.
- Great for onboarding onto a new project

### `/fg-from-docs`
Convert existing documentation (markdown files, wikis, Confluence exports) into file graph nodes. Extracts structure, cross-references, and metadata.

- Arguments: `$1` = docs source path

### `/fg-export`
Export the graph to other formats:
- GraphViz DOT
- Mermaid diagram syntax
- JSON adjacency list
- CSV edge list
- Arguments: `$1` = format, optional `$2` = subtree

---

## Workflow Integration

### `/fg-impact`
Given a node, analyze what other nodes would be affected by changes. Walks the edge graph transitively to show the blast radius.

- Arguments: `$1` = node path, optional `$2` = depth
- Useful before making changes to understand downstream effects

### `/fg-pr-context` (Background Skill)
When working on PRs, automatically identifies relevant graph nodes based on the files being changed and pulls them in as context. Non-user-invocable — triggers when PR-related work is detected.

### `/fg-sync`
After codebase changes, update the corresponding file graph nodes. Compares the current code state against what the graph describes and suggests updates.

- Pairs with `/fg-from-codebase` — one bootstraps, the other maintains

### `/fg-diff`
Show what changed in the graph between two Git refs. Not just file diffs — structural changes: new nodes, removed nodes, changed edges, new connections.

- Arguments: `$1` = base ref (default: `HEAD~1`), optional `$2` = compare ref (default: `HEAD`)

---

## Implementation Notes

- Skills should reference `${CLAUDE_SKILL_DIR}` for helper scripts (validation, traversal, index building)
- Use `` !`command` `` syntax for dynamic context injection where pre-computation helps (stats, validation)
- The `/fg-context` background skill should include the full protocol spec so any agent harness always has it available
- Skills that modify the graph (`fg-add`, `fg-link`, `fg-remove`, `fg-move`, `fg-ingest`) should use `disable-model-invocation: true` to prevent accidental triggering
- All skills should work with relative paths from the graph root

## Priority Order for Implementation

If building these incrementally, this order gives the most value fastest:

1. **`/fg-context`** — makes every manual interaction with the graph better immediately
2. **`/fg-add`** and **`/fg-link`** — basic graph building
3. **`/fg-validate`** — catch issues early
4. **`/fg-find`** and **`/fg-neighbors`** — navigation
5. **`/fg-from-codebase`** — big bang graph generation
6. **`/fg-visualize`** — seeing is believing
7. Everything else based on need
