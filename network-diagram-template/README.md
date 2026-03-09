# Network Diagram Template Kit

Enterprise network architecture diagrams (draw.io) generated from structured YAML input via Claude Code.

## Files

| File | Role |
|------|------|
| `PROMPT.md` | Generation rules for Claude Code (visual conventions, layout, architectural correctness) |
| `input.yaml` | Blank input template — copy and fill in |
| `input-example.yaml` | Filled-in example for reference |
| `output.drawio` | Sample output generated from the example |

## Usage

```bash
# 1. Copy and fill in
cp input.yaml my-network.yaml
vi my-network.yaml            # Replace all "???" / delete unused sections

# 2. Generate (in Claude Code)
#    "Read PROMPT.md and generate output.drawio from my-network.yaml"

# 3. View
code output.drawio            # VS Code + hediet.vscode-drawio extension
```

## Output

A `.drawio` file with 4 tabs and clickable drill-down navigation:

```
L1 Overview (hub-and-spoke)
├── Click AWS    → L2 AWS Detail (VPCs / subnets / services)
├── Click Azure  → L2 Azure & Alibaba Detail
├── Click DC     → L2 DC & Overseas Detail
└── Legend + connection types
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) for diagram generation
- [draw.io](https://github.com/jgraph/drawio-desktop/releases) for viewing (Desktop, VS Code extension, or [web](https://app.diagrams.net))
