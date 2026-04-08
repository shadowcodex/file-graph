# File Graph

A persistent, file-based knowledge graph that stores information in plaintext `.md` files. No databases. No embedding pipelines. No vector stores. Just files, frontmatter, and Git.

## The Idea

Agent harnesses (Claude Code, Codex, Antigravity, Open Code, etc.) are getting really good at navigating filesystems. So instead of building complex RAG infrastructure, let's just... put the knowledge graph on the filesystem and let agents do what they're already good at.

Let *them* invest in better file discovery. We invest in clean, structured data.

---

## Protocol Spec

### Nodes

A node is a Markdown file. That's it.

#### Node Type

A node's type is determined by its path relative to the graph root directory:

```
graph-root/
├── jira/issues/PROJ-123.md          ← type: jira/issues
├── jira/issues/PROJ-456.md          ← type: jira/issues
├── product/ui/available_forms.md     ← type: product/ui
├── product/api/endpoints.md          ← type: product/api
└── people/engineers/alice.md         ← type: people/engineers
```

The directory path *is* the type system. No enums, no config files, no type registries. Create a folder, put files in it — you've got a new type.

#### Node Structure

Every node file follows this format:

```markdown
---
name: Human Readable Name
description: Quick description for context discovery
explicit_edges:
  "path/to/related/doc.md": "Why this relationship exists and any edge metadata"
  "path/to/another/doc.md": "Description of this connection"
---

# Content goes here

Standard markdown. Write whatever you want.
```

**Required frontmatter fields:**

| Field | Purpose |
|-------|---------|
| `name` | Human-readable display name for the node |
| `description` | Short summary — this is what agents read first when deciding if a node is relevant |

**Recommended frontmatter fields:**

| Field | Purpose |
|-------|---------|
| `explicit_edges` | Map of `path: edge_description` pairs defining typed relationships to other nodes |
| `created_at` | ISO-8601 date when the node was first created (e.g., `"2026-04-08"`) |
| `updated_at` | ISO-8601 date when the node was last modified (e.g., `"2026-04-08"`) |
| `updated_by` | Who or what last modified the node — agent name, person, or process (e.g., `"analyst"`, `"linker"`, `"alice"`) |

**Other optional frontmatter fields:**

| Field | Purpose |
|-------|---------|
| anything else | Add whatever fields make sense for your use case — `status`, `priority`, etc. The spec doesn't care. |

---

### Edges

Edges connect nodes. There are two kinds:

#### 1. Explicit Edges

Defined in a node's frontmatter under `explicit_edges`. These are the "important" relationships — the ones you want to be obvious and discoverable.

```yaml
explicit_edges:
  "product/ui/available_forms.md": "This issue tracks a bug in the forms UI"
  "people/engineers/alice.md": "Assigned to Alice"
```

The value string is freeform. Use it to describe the relationship, store metadata, whatever you need. The edge info lives *on the edge*, not on either node.

**Directionality:** Edges are as directional as you make them. If node A has an explicit edge to node B, that's a one-way edge. If B also edges back to A, now it's bidirectional. The spec doesn't enforce either — it's a graph, do what makes sense. If you want to extend the protocol with formal directionality semantics, go for it.

#### 2. Inline Edges

References to other nodes within the markdown content body. Two formats supported:

**Wikilinks (preferred for graph tools like Obsidian):**
```markdown
This relates to [[product/ui/available_forms.md|Available Forms]]
```
Format: `[[<path>|<alias>]]`

**Standard Markdown links:**
```markdown
This relates to [Available Forms](product/ui/available_forms.md)
```
Format: `[alias](path)`

Both are valid. Use whatever your tooling prefers. Wikilinks are more natural for graph-native tools; markdown links are more portable.

**All paths are relative to the graph root directory.** Whether in `explicit_edges`, wikilinks, or markdown links — always reference nodes from the root.

**Overlap is fine.** If the same relationship shows up as both an explicit edge and an inline link, who cares? It's still just an edge. The explicit edge has its description, the inline link has its alias — combine them and move on. Let the agent figure it out.

---

### Content

Content is standard Markdown, but the protocol defines conventions for structured data:

#### Tabular / CSV Data

Wrap in a fenced `csv` code block:

```csv
header,row,column,names
1,2,3,4
data,in,the,row
```

A node storing tabular data is just a `.md` file with frontmatter + a CSV code block. Simple.

#### JSON Data

```json
{
  "key": "value",
  "nested": { "works": "fine" }
}
```

#### YAML Data

```yml
key: value
nested:
  works: fine
```

#### Any Other Format

The pattern extends to anything. Frontmatter for metadata, fenced code block for the payload. Store protobuf schemas, SQL, HCL, whatever — just label the code block appropriately.

---

### Tags

Nodes can have tags — standard hashtag format, placed anywhere in the markdown content:

```markdown
#activity #project #bug
```

Tags support hierarchy with `/` separators:

```markdown
#business/domain #product/frontend/forms
```

Tags are useful for cross-cutting concerns that don't fit neatly into the directory-based type system. A node can be `type: jira/issues` (by path) and also tagged `#frontend #p0 #sprint-42`.

Tags live inline in the content — deliberately *not* in frontmatter. Keeps things simple and readable. Also don't fall into the trap on just listing a bunch of tags at the end of a document, weave them into the content to make them relatable on why they are there.

---

### Sources

Nodes should include a `## Sources` section listing the materials that informed the document. This makes every claim traceable and helps readers (human or agent) evaluate trustworthiness.

```markdown
## Sources

- `src/api/handlers.py` — endpoint definitions and route structure
- `docs/api-reference.md` — official API documentation
- PROJ-1234 (Jira) — requirements for the v2 migration
- https://docs.vendor.com/api — third-party API reference
```

Sources can be file paths, ticket IDs, URLs, or any other reference. Each entry should briefly describe what information was drawn from that source.

Place the Sources section near the bottom of the document, before the Changelog.

---

### Changelog

Nodes should include a `## Changelog` table at the very bottom of the document. This provides a running history of when the node was created and modified.

```markdown
## Changelog

| Date | Author | Change |
|------|--------|--------|
| 2025-03-16 | alice | Added reproduction steps and linked blocker |
| 2025-03-15 | analyst | Initial creation |
```

The most recent entry goes first. The `Date` column uses ISO-8601 format. The `Author` column matches the `updated_by` frontmatter field. The `Change` column is a brief description of what changed.

On initial creation, the changelog has a single entry. Each subsequent modification adds a row.

---

### Full Node Example

```markdown
---
name: "Forms Validation Bug"
description: "Validation fails silently on the new patient intake form when required fields are left empty"
created_at: "2025-03-15"
updated_at: "2025-03-16"
updated_by: "alice"
explicit_edges:
  "product/ui/available_forms.md": "Bug is in the patient intake form component"
  "people/engineers/alice.md": "Assigned to Alice for sprint 42"
  "jira/issues/PROJ-400.md": "Blocked by this issue — needs API fix first"
---

# Forms Validation Bug

This #bug in the #frontend is a #p0 — the new patient intake form doesn't show validation errors when required fields are empty. Users can submit the form with missing data, which causes downstream errors in [[product/api/endpoints.md|the intake API]].

## Reproduction Steps

1. Navigate to /intake/new
2. Leave all fields empty
3. Click Submit
4. No errors shown, but the API returns 422

## Notes

Discussed in standup on 2025-03-15. Alice is picking this up after [[jira/issues/PROJ-400.md|PROJ-400]] lands.

## Sources

- `product/ui/available_forms.md` — form component inventory and validation logic
- PROJ-399 (Jira) — original bug report from QA team
- Standup notes 2025-03-15 — triage discussion and assignment

## Changelog

| Date | Author | Change |
|------|--------|--------|
| 2025-03-16 | alice | Added reproduction steps and linked blocker PROJ-400 |
| 2025-03-15 | explorer | Initial creation from Jira triage |
```

---

## Example Graph

Generated completely using claude code with phrase "Make some example graphs forp people to understand structure" and a few things of feedback like "Yo, don't put tags all at the bottom of the document weave them inline". 

If something is real here, its cause claude code made it... Sorry. Submit a PR to change it if you want.

### Solar System (`examples/solar-system/`)

A simple starter example — stars, planets, moons, and space missions. 9 nodes, ~20 edges, 4 node types. Good for understanding the format quickly.

```
examples/solar-system/
├── stars/sun.md
├── planets/{earth,mars,jupiter}.md
├── moons/{luna,europa,phobos}.md
└── missions/{apollo-11,curiosity}.md
```

### Acme SaaS (`examples/acme-saas/`)

A realistic corporate knowledge graph for a fictional B2B SaaS company. 18 nodes across 7 node types — teams, people, product features, incidents, customers, projects, and runbooks. The nodes are densely interconnected the way real organizational knowledge actually is: an incident links to the people who responded, the systems that broke, the customer who was impacted, and the runbook that was written afterward.

```
examples/acme-saas/
├── teams/{platform,product}.md
├── people/{sarah-chen,marcus-jones,priya-sharma,alex-kim,jordan-taylor}.md
├── products/features/{api-gateway,data-pipeline,dashboard,billing}.md
├── incidents/2025-02-14-pipeline-outage.md
├── customers/{meridian-health,northwind-logistics,cascade-analytics}.md
├── projects/{auth-migration,usage-analytics}.md
└── runbooks/pipeline-backpressure.md
```

---

## Storage

### Git (Start Here)

Store the graph in a Git repo. You get versioning, collaboration, branching, and diffing for free. The graph *is* the repo.

```
my-knowledge-graph/          ← this is a git repo
├── jira/issues/
├── product/ui/
├── product/api/
├── people/engineers/
└── ...
```

### Cloud Storage (If You Must)

If you outgrow Git (large binary payloads, extremely high write volume), you can graduate to GCS, S3, or whatever. But seriously — start with Git.

---

## Retrieval

Three ways to get the graph (or parts of it) where you need it:

### 1. Clone

```sh
git clone <repository-url>
```

Full graph, full history. Tell your agent harness where it lives and let it explore.

### 2. Sparse Checkout

Grab specific subtrees with their Git history:

```sh
git init <repo-name>
cd <repo-name>
git remote add origin <repository-url>
git sparse-checkout init --cone
git sparse-checkout set product/ui people/engineers
git pull
```

Great for when you only need a slice of a large graph but still want to commit changes back.

### 3. Git Archive

Download a folder snapshot — no Git history, no `.git` directory:

```sh
git archive --remote=<repository-url> HEAD:product/ui | tar -x
```

Good for read-only use cases or feeding a subtree into a process.

**Pick whichever fits your workflow.** If you're editing the graph and pushing changes, use clone or sparse-checkout. If you're just reading, archive is lighter.

---

## Indexing

Index files can live at directory roots to provide pre-computed summaries — tag indexes, node counts, relationship maps. These are optional convenience files, not part of the core protocol.

We'll likely provide CLI utilities or skills for generating these. Or maybe Obsidian can handle it. Or maybe you just ask Claude Code to build one. The filesystem is the source of truth either way.

---

## Tooling Compatibility

### Obsidian.md

This format is highly compatible with [Obsidian.md](https://obsidian.md/). Obsidian natively understands YAML frontmatter, wikilinks, tags, and markdown content. You can use it to browse, edit, and visualize the graph.

You might need a lightweight Obsidian plugin to fully leverage `explicit_edges` from frontmatter — but honestly, you could probably just ask Claude Code to write that plugin for you.

### Agent Harnesses

The whole point. Claude Code, Codex, Antigravity, Open Code, and other agent harnesses can navigate, read, and modify the graph using their built-in filesystem tools. No special integrations needed.

### Claude Code Skills

We provide skills ideas (in the `skills/` directory of this repo) designed for generating, maintaining, and working with file graphs in Claude Code. See `skills/` for details.

---

## Design Principles

1. **Files are the API.** If you can read a file, you can read the graph. No query language, no client library, no server.
2. **The filesystem is the type system.** Directory paths define node types. No schemas to maintain.
3. **Edges are just references.** Explicit edges in frontmatter for important relationships, inline links for casual ones.
4. **Git is the database.** Versioning, collaboration, and distribution come free.
5. **Agents figure it out.** Don't over-engineer discovery. Agent harnesses are getting better at filesystem navigation every day. Invest in clean data, not complex retrieval infrastructure.
6. **Plaintext is forever.** No proprietary formats, no binary blobs (for the graph itself), no lock-in. It's Markdown files in a Git repo. It'll outlive whatever tools we're using today.
