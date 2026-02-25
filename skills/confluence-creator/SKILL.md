---
name: confluence-creator
description: Create comprehensive, professionally-designed Confluence pages with embedded draw.io architecture diagrams. Researches content from available MCPs (Atlassian, ServiceNow, GitHub, etc.), generates validated draw.io diagrams with proper connectors, and publishes polished pages. Use when user wants to create architecture docs, design documents, or any rich Confluence page with diagrams.
license: MIT
compatibility: Requires atlassian MCP. Optional MCPs auto-discovered at runtime.
metadata:
  author: Tim Schwarz
  version: "1.0"
---

# Confluence Page Creator

Creates professional Confluence pages with embedded draw.io architecture diagrams. Researches content from all available MCPs, generates validated diagrams, and publishes everything as a polished, interlinked page.

## When to Use

- User asks to create a Confluence page, architecture document, or design doc
- User asks to document a system, service, or integration on Confluence
- User says "create a confluence page for X" or "document X on Confluence"
- User needs draw.io diagrams embedded in Confluence

## Execution Steps

### Phase 1: Discovery & Setup

1. **Connect the Atlassian MCP** — this is required:
```
mcpu-mux(["connect", "atlassian"])
```

2. **Discover all available MCPs** for content research:
```
mcpu-mux(["servers"])
```
Connect any MCPs relevant to the topic (e.g., `servicenow` for ITSM docs, `github` for code references, `dynatrace` for observability). The more sources, the richer the document.

3. **Clarify the target** with the user using `ask_user`:
   - What topic / system / service to document?
   - Which Confluence space and parent page? (or create new?)
   - What audience? (engineering, leadership, security, all)
   - What emphasis? (security, architecture, operations, all)
   - Any existing pages to update vs. create fresh?

### Phase 2: Research

4. **Search Confluence** for existing content on the topic. Run MULTIPLE search queries with varied terms:
```
mcpu-mux(["call", "atlassian", "confluence_search"], { query: "TOPIC_TERM_1", limit: 20 })
mcpu-mux(["call", "atlassian", "confluence_search"], { query: "TOPIC_TERM_2", limit: 20 })
mcpu-mux(["call", "atlassian", "confluence_search"], { query: "TOPIC_TERM_3", limit: 20 })
```
For each relevant result, get the full page:
```
mcpu-mux(["call", "atlassian", "confluence_get_page"], { page_id: "ID", convert_to_markdown: true })
```

5. **Search Jira** for related tickets, epics, and decisions:
```
mcpu-mux(["call", "atlassian", "jira_search"], { jql: "text ~ \"TOPIC\" AND updated >= \"DATE\" ORDER BY updated DESC", limit: 50 })
```

6. **Search other connected MCPs** as appropriate:
   - ServiceNow: incidents, changes, CIs related to the topic
   - GitHub: repos, PRs, code related to the topic
   - Any other connected MCP with relevant data

7. **Compile research** into a structured findings file at `/tmp/confluence-creator-research.md`. Organize by source.

### Phase 3: Content Generation

8. **Generate the page content** as clean markdown. Follow these writing rules:

   **Structure:**
   - Start with Executive Summary (no "Document Control" section — Confluence handles metadata)
   - Use numbered sections: §1, §2, §3...
   - Use tables extensively for structured data
   - Use blockquotes with `> **Warning:**`, `> **Note:**`, `> **Info:**` prefixes (NO emoji in headers)
   - End with References section combining related docs + Jira tracking

   **Tone:**
   - Professional, direct, active voice
   - Cut filler: no "It is important to note", "This ensures that", "This approach"
   - Say it once, clearly — don't restate what tables already show
   - Bullet points over paragraphs when content is a list of facts

   **References:**
   - Include Confluence page IDs for cross-references
   - Include Jira ticket keys (PROJ-XXXX) for traceability
   - Link to related pages in other spaces

9. **Save content** to `/tmp/confluence-creator-content.md`

### Phase 4: Draw.io Diagram Generation

10. **Generate draw.io diagrams** as separate files. Each diagram is a single-page `.drawio` file.

    **CRITICAL — Follow the Draw.io Connector Rules (see below)**

    Typical diagram set for an architecture doc:
    - `01-Architecture-Overview.drawio` — high-level component/zone view
    - `02-Data-Flow.drawio` — sequence or flow diagram
    - `03-Integration-Map.drawio` — integration points with protocols
    - Additional diagrams as needed for the topic

    Save to: `/tmp/confluence-creator-diagrams/`

11. **Validate ALL diagrams** using the connector validation script (see below). Fix any issues before proceeding.

### Phase 5: Publish

12. **Create or update the Confluence page** with markdown content:
```
mcpu-mux(["call", "atlassian", "confluence_create_page"], {
  space_key: "SPACE",
  title: "Page Title",
  content: MARKDOWN_CONTENT,
  parent_id: "PARENT_PAGE_ID",  // optional
  content_format: "markdown",
  enable_heading_anchors: true
})
```
Or for updates:
```
mcpu-mux(["call", "atlassian", "confluence_update_page"], {
  page_id: "PAGE_ID",
  title: "Page Title",
  content: MARKDOWN_CONTENT,
  content_format: "markdown",
  version_comment: "Description of changes",
  enable_heading_anchors: true
})
```

13. **Upload draw.io files as attachments** — the user must do this manually through the Confluence UI (drag and drop the `.drawio` files onto the page), OR if they're already attached, proceed to embedding.

14. **Embed draw.io diagrams** into the page using storage format injection:

    a. Get the current storage format:
    ```
    mcpu-mux(["call", "atlassian", "confluence_get_page"], {
      page_id: "PAGE_ID", convert_to_markdown: false, include_metadata: false
    })
    ```

    b. For each diagram, create a drawio macro and inject it at the correct section:
    ```xml
    <ac:structured-macro ac:local-id="GENERATE_UUID" ac:macro-id="GENERATE_UUID"
      ac:name="drawio" ac:schema-version="1" data-layout="default">
      <ac:parameter ac:name="mVer">2</ac:parameter>
      <ac:parameter ac:name="simple">0</ac:parameter>
      <ac:parameter ac:name="zoom">1</ac:parameter>
      <ac:parameter ac:name="inComment">0</ac:parameter>
      <ac:parameter ac:name="pageId">PAGE_ID</ac:parameter>
      <ac:parameter ac:name="diagramDisplayName">FILENAME.drawio</ac:parameter>
      <ac:parameter ac:name="lbox">1</ac:parameter>
      <ac:parameter ac:name="revision">1</ac:parameter>
      <ac:parameter ac:name="baseUrl">https://qurate.atlassian.net/wiki</ac:parameter>
      <ac:parameter ac:name="diagramName">FILENAME.drawio</ac:parameter>
      <ac:parameter ac:name="pCenter">0</ac:parameter>
      <ac:parameter ac:name="width">1596</ac:parameter>
      <ac:parameter ac:name="links"></ac:parameter>
      <ac:parameter ac:name="tbstyle"></ac:parameter>
      <ac:parameter ac:name="height">897</ac:parameter>
    </ac:structured-macro>
    ```

    c. Find the target section heading in the storage XHTML (search for the anchor `ac:name` matching the section slug), then insert the macro right after the closing `</h2>` or `</h3>` tag.

    d. Re-publish the page with `content_format: "storage"`:
    ```
    mcpu-mux(["call", "atlassian", "confluence_update_page"], {
      page_id: "PAGE_ID",
      title: "Page Title",
      content: UPDATED_STORAGE_XHTML,
      content_format: "storage",
      version_comment: "Embedded draw.io diagrams"
    })
    ```

15. **Report completion** to the user with:
    - Confluence page URL
    - List of embedded diagrams
    - List of diagram files saved locally (for editing in draw.io desktop)
    - Any open items or warnings

---

## Draw.io Connector Rules

**This is the most critical section.** Bad connectors make diagrams look unprofessional. Every connector MUST be validated before publishing.

### Rule 1: Every Edge Must Have source AND target

Every `<mxCell>` that is an edge (has `edge="1"` or an `edgeStyle`/`endArrow` in its style) MUST have both `source="ID"` and `target="ID"` attributes pointing to valid cell IDs that exist in the diagram.

```xml
<!-- ✅ CORRECT — connected to real cells -->
<mxCell id="200" edge="1" source="110" target="120"
  style="edgeStyle=orthogonalEdgeStyle;rounded=1;endArrow=block;"
  parent="1">
  <mxGeometry relative="1" as="geometry"/>
</mxCell>

<!-- ❌ WRONG — no source or target -->
<mxCell id="200" edge="1"
  style="edgeStyle=orthogonalEdgeStyle;endArrow=block;"
  parent="1">
  <mxGeometry as="geometry">
    <mxPoint x="200" y="300" as="sourcePoint"/>
    <mxPoint x="500" y="300" as="targetPoint"/>
  </mxGeometry>
</mxCell>
```

**Never use `sourcePoint`/`targetPoint` coordinates for connectors between components.** These create floating lines that don't move when components are repositioned. Always use `source="ID"` and `target="ID"`.

### Rule 2: Anchor Points Must Be Paired

If a style has `exitX`/`exitY` (where the line leaves the source), it MUST also have `entryX`/`entryY` (where the line enters the target), and vice versa. Unpaired anchors cause lines to connect at wrong positions.

```
<!-- ✅ CORRECT — both exit and entry anchors -->
style="exitX=1;exitY=0.5;exitDx=0;exitDy=0;entryX=0;entryY=0.5;entryDx=0;entryDy=0;"

<!-- ❌ WRONG — exit but no entry -->
style="exitX=1;exitY=0.5;exitDx=0;exitDy=0;"
```

Standard anchor positions:
- **Right edge center**: `X=1;Y=0.5` — use for left-to-right flows
- **Left edge center**: `X=0;Y=0.5` — use for incoming connections
- **Top center**: `X=0.5;Y=0` — use for top-down flows
- **Bottom center**: `X=0.5;Y=1` — use for downward connections

### Rule 3: Use orthogonalEdgeStyle for Architecture Diagrams

Architecture diagrams must use clean right-angle routing, not curved or direct lines.

```
style="edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=auto;html=1;
  strokeColor=#333333;strokeWidth=2;endArrow=block;endFill=1;
  exitX=1;exitY=0.5;exitDx=0;exitDy=0;
  entryX=0;entryY=0.5;entryDx=0;entryDy=0;"
```

Key style properties for clean connectors:
- `edgeStyle=orthogonalEdgeStyle` — right-angle routing
- `rounded=1` — rounded corners on bends
- `orthogonalLoop=1` — proper loop routing
- `jettySize=auto` — automatic spacing from shapes
- `endArrow=block;endFill=1` — solid arrowhead
- `strokeWidth=2` — visible line weight

### Rule 4: Label Placement

Edge labels must use `<mxGeometry>` with `relative="1"` and an `<mxPoint>` for offset:

```xml
<mxCell id="200" value="OAuth2/OIDC" edge="1" source="110" target="120"
  style="edgeStyle=orthogonalEdgeStyle;rounded=1;endArrow=block;endFill=1;
    strokeColor=#CC0000;strokeWidth=2;fontColor=#CC0000;fontSize=10;
    exitX=1;exitY=0.5;exitDx=0;exitDy=0;
    entryX=0;entryY=0.5;entryDx=0;entryDy=0;"
  parent="1">
  <mxGeometry relative="1" as="geometry">
    <mxPoint y="-10" as="offset"/>
  </mxGeometry>
</mxCell>
```

### Rule 5: Consistent Color Coding for Arrows

Use color to convey meaning:
- **Red** (`#CC0000`) — authentication / security flows
- **Blue** (`#0066CC`) — data flows
- **Green** (`#006600`) — management / control flows
- **Orange** (`#FF6600`) — external / third-party flows
- **Gray** (`#666666`) — internal / infrastructure flows

### Connector Validation Script

After generating ANY draw.io file, ALWAYS run this validation:

```python
import re, sys, os

def validate_drawio(filepath):
    with open(filepath) as f:
        content = f.read()

    errors = []
    warnings = []

    # Collect all cell IDs
    all_ids = set(re.findall(r'id="([^"]+)"', content))

    # Find all edges
    edge_pattern = re.compile(
        r'<mxCell\s+id="([^"]+)"[^>]*?'
        r'(?:edge="1"|style="[^"]*(?:endArrow|edgeStyle)[^"]*")'
        r'[^>]*?>',
        re.DOTALL
    )

    for match in edge_pattern.finditer(content):
        cell_text = match.group()
        cell_id = match.group(1)

        # Check source and target
        source = re.search(r'source="([^"]+)"', cell_text)
        target = re.search(r'target="([^"]+)"', cell_text)

        if not source:
            errors.append(f"Edge {cell_id}: MISSING source attribute")
        elif source.group(1) not in all_ids:
            errors.append(f"Edge {cell_id}: source '{source.group(1)}' does not exist")

        if not target:
            errors.append(f"Edge {cell_id}: MISSING target attribute")
        elif target.group(1) not in all_ids:
            errors.append(f"Edge {cell_id}: target '{target.group(1)}' does not exist")

        # Check anchor pairing
        style = re.search(r'style="([^"]*)"', cell_text)
        if style:
            s = style.group(1)
            has_exit = 'exitX=' in s
            has_entry = 'entryX=' in s
            if has_exit and not has_entry:
                warnings.append(f"Edge {cell_id}: has exitX/Y but missing entryX/Y")
            if has_entry and not has_exit:
                warnings.append(f"Edge {cell_id}: has entryX/Y but missing exitX/Y")

        # Check for sourcePoint/targetPoint (bad practice)
        geo_end = content.find('</mxCell>', match.end())
        geo_block = content[match.end():geo_end] if geo_end > 0 else ''
        if 'sourcePoint' in geo_block and not source:
            warnings.append(f"Edge {cell_id}: uses sourcePoint instead of source attribute")
        if 'targetPoint' in geo_block and not target:
            warnings.append(f"Edge {cell_id}: uses targetPoint instead of target attribute")

    return errors, warnings

# Run validation
diagrams_dir = '/tmp/confluence-creator-diagrams'
all_pass = True
for fname in sorted(os.listdir(diagrams_dir)):
    if not fname.endswith('.drawio'):
        continue
    path = os.path.join(diagrams_dir, fname)
    errors, warnings = validate_drawio(path)
    status = "✅ PASS" if not errors else "❌ FAIL"
    print(f"{status} {fname}")
    for e in errors:
        print(f"  ❌ {e}")
        all_pass = False
    for w in warnings:
        print(f"  ⚠️  {w}")

if not all_pass:
    print("\n❌ VALIDATION FAILED — fix all errors before publishing")
    sys.exit(1)
else:
    print("\n✅ All diagrams passed validation")
```

**If validation fails, DO NOT publish.** Fix the draw.io XML and re-validate until all errors are resolved. Warnings should be addressed but are not blockers.

### Fixing Common Connector Issues

**Missing source/target:** Find the cell IDs of the shapes the edge should connect, then add `source="ID" target="ID"` to the `<mxCell>` tag.

**Floating lines (sourcePoint/targetPoint only):** Replace with proper `source`/`target` attributes. Remove the `<mxPoint>` sourcePoint/targetPoint entries from the geometry.

**Unpaired anchors:** Add the missing exit or entry anchor coordinates. Use the standard positions (Rule 2).

**Lines routing through shapes:** Add `jettySize=auto` and ensure `orthogonalLoop=1` is in the style.

---

## Draw.io Design Standards

### Layout & Spacing
- Page size: minimum 1400×900px
- Component spacing: 40px minimum between shapes
- Zone/group padding: 20px inside borders
- Consistent alignment: use grid (10px)

### Typography
- Font: Helvetica
- Component labels: 11pt, bold
- Zone titles: 12pt, bold
- Edge labels: 10pt
- Title block: 18pt bold (page title), 13pt (subtitle)

### Shapes
- Components: `rounded=1;whiteSpace=wrap;html=1;` (rounded rectangles)
- Data stores: `shape=cylinder3;` (cylinders)
- External services: `shape=cloud;` or `ellipse;shape=cloud;`
- Security zones: `dashed=1;strokeWidth=2;` (dashed borders with zone-specific colors)
- Users/actors: `shape=mxgraph.basic.person;` or simple labeled rectangle

### Zone Colors (for architecture diagrams)
- External: `fillColor=#FFE8E8;strokeColor=#CC0000` (red)
- Edge/CDN: `fillColor=#FFF0E0;strokeColor=#FF6600` (orange)
- Identity/Auth: `fillColor=#E0EFFF;strokeColor=#0066CC` (blue)
- Gateway: `fillColor=#E0FFE0;strokeColor=#006600` (green)
- Application: `fillColor=#F0E8FF;strokeColor=#6600CC` (purple)
- Data: `fillColor=#F0F0F0;strokeColor=#666666` (gray)

---

## Critical Rules

1. **ALWAYS connect the Atlassian MCP first** — the skill cannot function without it
2. **ALWAYS validate draw.io connectors** before publishing — run the validation script on every diagram
3. **NEVER use sourcePoint/targetPoint for component connectors** — always use `source="ID" target="ID"`
4. **NEVER publish diagrams that fail validation** — fix first, then publish
5. **ALWAYS use `content_format: "storage"` for the final publish** when embedding draw.io macros — markdown format cannot include `ac:structured-macro` elements
6. **ALWAYS generate UUIDs** for `ac:local-id` and `ac:macro-id` in drawio macros — never reuse IDs
7. **ALWAYS save diagrams locally** to `~/Desktop/` or a user-specified location so they can edit in draw.io desktop
8. **ALWAYS provide the page URL** to the user after publishing
9. **Research from MULTIPLE sources** — don't rely on a single search query; use varied terms and multiple MCPs
10. **Professional tone only** — no emoji in headers, no filler phrases, active voice, direct language
