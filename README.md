# Architecture Diagram Generator

Describe a system in plain words and the app draws its architecture diagram for you.

For example, type:

> Trading workflow where a user submits an order, the risk engine validates it, and the execution venue confirms it

and you get a flowchart with the steps, decisions and databases in it, which you can then improve and edit.

## How it works

1. **Understands your description.** Vague or non-English text is rewritten into a clear description.
2. **Plans the diagram.** It finds the components, how they connect, the decision points, and important pieces you forgot to mention.
3. **Draws it.** The diagram is generated as [Mermaid](https://mermaid.js.org/) code and shown on a canvas you can zoom and drag.
4. **Helps you improve it.** It suggests improvements you can preview and accept, and you can ask for changes in plain words ("add an error path from the risk engine"). Any change can be undone.

## Built with

Python · Streamlit · LangChain · OpenAI / Anthropic models · Mermaid · Docker

## Run it

You need [uv](https://docs.astral.sh/uv/) and an OpenAI API key.

```bash
mkdir -p .streamlit
echo 'OPENAI_API_KEY = "your_api_key"' > .streamlit/secrets.toml
uv run streamlit run main.py
```

Then open http://localhost:8501.
