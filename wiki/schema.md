# Wiki Schema

Conventions and workflows for this wiki. Read this before editing any page.

## Directory Layout

```
wiki/
├── index.md          # Master catalog — update on every change
├── log.md            # Append-only activity log
├── schema.md         # This file — conventions and workflows
├── *.md              # Topic pages (architecture, routing, etc.)
└── code/             # Hierarchical code summaries
    ├── index.md
    ├── shared/       # manifest-shared package
    │   ├── index.md
    │   └── {module}.md
    ├── backend/      # manifest-backend package
    │   ├── index.md
    │   └── {module}.md
    ├── frontend/     # manifest-frontend package
    │   ├── index.md
    │   └── {module}.md
    └── manifest/     # version shell package
        └── index.md
```

## Page Conventions

- **Frontmatter**: optional YAML block at top for tags/dates if using Dataview
- **First paragraph**: one-sentence summary of what the page covers
- **Cross-links**: always use relative paths — `[routing](routing.md)`, `[proxy module](code/backend/routing.md)`
- **Code snippets**: use fenced blocks with language tag
- **"See also"**: end every page with a `## See Also` section listing 3-5 related pages
- **Headings**: H1 = page title, H2 = major sections, H3 = subsections

## Code Wiki Conventions

- **`code/{package}/index.md`**: package overview, responsibility, dependencies, key exports, module map
- **`code/{package}/{module}.md`**: single module/directory — what it does, key files, public API, gotchas
- Module names match the directory or controller name (e.g. `routing.md` for `src/routing/`, `auth.md` for `src/auth/`)

## Workflows

### Ingest a new source
1. Read the source
2. Create or update the relevant topic page(s)
3. Update `index.md` if a new page was created
4. Append an entry to `log.md`
5. Update any code wiki pages touched by the change

### Answer a query
1. Read `index.md` to find relevant pages
2. Read those pages, synthesize an answer
3. If the answer is novel and reusable, file it as a new page or append to an existing one
4. Log the query in `log.md` if it produced a new page

### Lint
Run periodically to keep the wiki healthy:
- Find pages with no inbound links (orphans)
- Flag claims contradicted by newer sources
- Identify important concepts mentioned but lacking their own page
- Check all cross-links resolve

## Source of Truth Hierarchy

For any conflict between this wiki and the actual code, **the code wins**. The wiki is derived; the code is authoritative. Flag contradictions in `log.md` with `## [DATE] contradiction | ...` and correct the wiki page.
