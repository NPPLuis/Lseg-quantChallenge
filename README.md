# Natural Language → Architecture Diagram Generator

Describe a system in plain language and get an editable flowchart of its architecture back.

Example prompt (one of the examples built into the app):

> Trading workflow where a user submits an order, the risk engine validates it, and the execution venue confirms it

## How it works

The app runs your description through a four-stage LLM pipeline built with LangChain:

```
 description ──► 1. Refine ──► 2. Analyze ──► 3. Generate ──► diagram ──► 4. Enhance
                 (rewrite)     (structured     (Mermaid        │           (suggestions,
                                 JSON)           code)          │            free-form edits,
                                                                └───────────  undo)
```

1. **Refine the prompt** (`src/preprocessing`). A vague, short or non-English description is rewritten into a precise English one that is easier to turn into a diagram.
2. **Analyze the architecture** (`src/analysis`). The model returns *structured output* validated with Pydantic: the components and the shape each should have (database → cylinder, decision → diamond, external service → subroutine, …), the relationships between them, decision points, best-practice notes, and up to three **missing components** the flow would break without.
3. **Generate the diagram** (`src/generation`). The description and the analysis are turned into a Mermaid flowchart, streamed live into the UI. The model is instructed to include every component from the analysis, use the assigned shapes, and color-code nodes by type (decisions, databases, start/end points, external services). The output is then normalized: code fences are stripped, the diagram type is checked against Mermaid's known types, and bare edge lists get a `flowchart` header.
4. **Enhance it** (`src/enhancement`). The model proposes up to five improvements (layout, labels, styling, completeness, best practices), each with its own preview. You pick the ones you want and they are merged into the diagram. You can also edit the diagram by typing an instruction ("add a retry path from the risk engine back to the trader"). Every edit is kept in a history and can be undone.

## Features

- Interactive diagram canvas: zoom with buttons or the mouse wheel, drag to pan
- Tabs for the diagram, the raw Mermaid code, the architecture analysis and the enhancement tools
- Generated diagrams are cached per prompt during a session, so repeating a prompt does not call the models again
- Works with OpenAI or Anthropic models, configurable separately for each pipeline stage
- Docker image with a health check

## Tech stack

Python 3.14 · Streamlit · LangChain · Pydantic · Mermaid.js · OpenAI / Anthropic APIs · uv · Docker

## Running locally

Requires [uv](https://docs.astral.sh/uv/) and an OpenAI API key. All stages use OpenAI by default; to use Anthropic models instead, add an Anthropic key and change the providers as shown below.

```bash
mkdir -p .streamlit
cat > .streamlit/secrets.toml <<'EOF'
OPENAI_API_KEY = "your_api_key"
# ANTHROPIC_API_KEY = "your_api_key"
EOF

uv run streamlit run main.py
```

The app opens at http://localhost:8501.

### Choosing models

Each stage reads its model from environment variables (or root-level keys in `.streamlit/secrets.toml`). The default is `gpt-5.4` from OpenAI everywhere.

| Stage | Variables |
|-------|-----------|
| Refine | `OPTIMIZER_MODEL`, `OPTIMIZER_PROVIDER` |
| Analyze | `ANALYZER_MODEL`, `ANALYZER_PROVIDER` |
| Generate | `MERMAID_AGENT_MODEL`, `MERMAID_AGENT_PROVIDER`, `MERMAID_AGENT_TEMPERATURE` |
| Enhance | `ENHANCEMENT_MODEL`, `ENHANCEMENT_PROVIDER`, `ENHANCEMENT_TEMPERATURE`, `ENHANCEMENT_MAX_SUGGESTIONS` |

For example, to run the generation stage on Claude (the other stages stay on OpenAI):

```toml
MERMAID_AGENT_PROVIDER = "anthropic"
MERMAID_AGENT_MODEL = "claude-sonnet-5-5"
```

### Docker

With `.streamlit/secrets.toml` in place:

```bash
docker build -t diagram-generator .
docker run -p 8501:8501 diagram-generator
```

The secrets file is copied into the image, so keep the image local and don't push it to a public registry.

## Project structure

```
main.py               Streamlit UI and pipeline orchestration
src/
  preprocessing/      stage 1: prompt refinement
  analysis/           stage 2: structured architecture analysis (Pydantic schemas)
  generation/         stage 3: Mermaid generation, validation and rendering
  enhancement/        stage 4: suggestions, free-form edits
data/initial.txt      sample input
Dockerfile            container image (uv + Streamlit)
```
