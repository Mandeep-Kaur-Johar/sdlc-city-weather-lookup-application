# Diagram Rendering Note - Epic 2327

Last verified: 2026-10-07.

The MCP Kroki/Mermaid PNG rendering service (`render_and_commit_architecture_diagrams`)
still returns a repeatable server-side error:

```text
cannot access local variable 'png_signature' where it is not associated with a value
```

Re-tested on 2026-10-07 with:

1. The full `system-context` payload.
2. A minimal three-node `flowchart TD` payload with default arguments.

Both attempts failed identically, confirming a renderer-service fault rather than a
Mermaid syntax fault. `output_format="svg"` is rejected by the same tool with
`Only png output is currently supported`.

## What is delivered instead

- All eight Mermaid sources are committed in this folder as `.mmd` files and are the
  authoritative, version-controlled form of every diagram.
- The System Design Document embeds each diagram inline as a fenced ```mermaid``` block,
  which renders natively in GitHub, so no diagram is missing from the document.
- PNG renderings were produced out-of-band and supplied directly to the requester.

## Committed diagram sources

| File | Diagram |
|---|---|
| `system-context.mmd` | System Context |
| `diagram-architecture.mmd` | Solution Architecture |
| `diagram-highlevel.mmd` | High-Level End-to-End Flow |
| `diagram-component.mmd` | Component Diagram |
| `diagram-datamodel.mmd` | Data Model (ER) |
| `diagram-deployment.mmd` | Deployment Topology |
| `diagram-sequence.mmd` | Critical Workflow Sequence |
| `diagram-cicd.mmd` | CI/CD Pipeline |

## Action required

Re-run `render_and_commit_architecture_diagrams` for Epic 2327 once the Kroki renderer
is healthy. The `.mmd` sources require no change; rendering them will produce the PNG
files and the document can then be regenerated with `diagram_mapping` populated.
