

## Directory 
```bash
src
 └── agentName # Same Directory where the command was executed, also show on UI
     ├── .env
     ├── __init__.py
     ├── agent.py # default file name defining the agent's logic. Must also contain a root_agent variable
     └── tools.py # optional, with all user defined tool functions
```
## Google Agent Development Kit (ADK)
`google.adk.cli` module (part of the `google-adk` package) is the command-line interface (CLI) entry point for the Google ADK, a framework for building, evaluating, and deploying AI agents. 
The `web` subcommand of `google.adk.cli` launches the ADK's built-in web-based developer UI (referred to as `adk web`). This UI is designed to simplify agent development and debugging by providing an interactive interface to test and inspect agents.

When executed, `adk web` starts a local web server (typically on http://localhost:8000 or http://127.0.0.1:8000) that hosts the ADK developer UI, allowing users to interact with their agents, view event logs, and test functionalities like text, voice, or video streaming.

The Google ADK is an open-source, modular framework for developing and deploying AI agents, optimized for Google’s Gemini models and the Google Cloud ecosystem but designed to be model-agnostic and deployment-agnostic. It simplifies the creation of intelligent agents that can perform tasks, use tools, and collaborate in multi-agent systems.

* ### Environment Setup by uv:
    * `uv run` ensures that the Python environment is correctly configured with the required version (Python 3.9+ for ADK) and dependencies (e.g., google-adk). If the google-adk package is not installed, uv can automatically install it, assuming it's specified in a pyproject.toml or similar configuration file.
    * It activates a virtual environment (or creates one if needed) to isolate dependencies, avoiding conflicts with other Python projects.
* ### Launching the Python Interpreter:
    * The `python -m google.adk.cli` part invokes the Python interpreter to execute the `google.adk.cli` module, which is the CLI component of the ADK.
* ### Executing the web Subcommand:
    * The `web` subcommand starts a local web server that hosts the ADK developer UI. This UI is built using Angular and requires Node.js, npm, and the google-adk Python package to be installed.
    * The UI allows you to interact with the agent via a chat interface, access to Trace/Events/State/Artifacts/Sessions/Eval, create/delete/export a session.
* ### Agent Configuration:
    * The agent's logic must be defined in an `agent.py` file within an agent directory (e.g., `multi_tool_agent/`), with a `root_agent` variable specifying the agent's properties (name, model, tools, instructions)
    * A .env file in the agent directory should contain necessary configurations (authentication), such as:
    ```bash
    GOOGLE_GENAI_USE_VERTEXAI=TRUE
    GOOGLE_API_KEY=your-api-key
    GOOGLE_CLOUD_PROJECT=your-project-id    # default if not specified
    GOOGLE_CLOUD_LOCATION=us-central1       # default if not specified
    ```
* ### Agent Configuration:
    * The `google-adk` package must be installed 
    * For the web UI, additional dependencies like Angular CLI, Node.js, npm and uvicorn/gradio may be required
    * If voice or video streaming is used, a Gemini model supporting the Live API must be specified in the agent.py file



## Example Workflow

#### 1. Set up the Project:
- Create a project directory with an agent folder (e.g., multi_tool_agent/)
- Define an `agent.py` file with a root_agent, such as
```python
# The LlmAgent (often aliased simply as Agent)
from google.adk.agents import Agent
from google.adk.tools import google_search

root_agent = Agent(
    name="search_assistant",
    model="gemini-2.0-flash",
    instruction="You are a helpful assistant. Answer user questions using Google Search when needed.",
    description="An assistant that can search the web.",
    tools=[google_search]
)
```
- `name` (Required): A unique string identifier for the agent
- `model` (Required): Specify the underlying LLM that will power this agent's reasoning.
- `instruction` parameter is arguably the most critical for shaping an LlmAgent's behavior. It's a string (or a function returning a string) that tells the agent:
    - Its core task or goal.
    - Its personality or persona (e.g., "You are a helpful assistant," "You are a witty pirate").
    - Constraints on its behavior (e.g., "Only answer questions about X," "Never reveal Y").
    - How and when to use its tools. You should explain the purpose of each tool and the circumstances under which it should be called, supplementing any descriptions within the tool itself.
    - The desired format for its output (e.g., "Respond in JSON," "Provide a bulleted list").

    The instruction is a string template, you can use the {var} syntax to insert dynamic values into the instruction. {var} is used to insert the value of the state variable named var. {artifact.var} is used to insert the text content of the artifact named var. If the state variable or artifact does not exist, the agent will raise an error. If you want to ignore the error, you can append a ? to the variable name as in {var?}.
- `description` (Optional, Recommended for Multi-Agent): Provide a concise summary of the agent's capabilities. This description is primarily used by other LLM agents to determine if they should route a task to this agent. Make it specific enough to differentiate it from peers.

- `tools` (Optional): Provide a list of tools the agent can use. Tools give your LlmAgent capabilities beyond the LLM's built-in knowledge or reasoning. They allow the agent to interact with the outside world, perform calculations, fetch real-time data, or execute specific actions.

- Advanced Configuration & Control  (`generate_content_config`, `input_schema`, `output_schema`, `output_key`, `include_contents`)

#### 2. Create a `.env` file with Google Cloud credentials or API keys
#### 3. Install Dependencies
```bash
# Install uv if not already installed
curl -LsSf https://astral.sh/uv/install.sh | sh
# Install google-adk and other dependencies
uv pip install google-adk
```

#### 4. Run the Command
```bash
# To run the agent via the web UI
uv run python -m google.adk.cli web
# To run the agent via CLI instead of the web UI
uv run python -m google.adk.cli run <agent_name>
# To start the API server for the web UI
uv run python -m google.adk.cli api_server --allow_origins=http://localhost:4200 --host=0.0.0.0
```
#### 5. Access the UI
- Open http://localhost:8000 in a browser.
- Select the agent (e.g., search_assistant) from the UI dropdown.
- Type a prompt (e.g., “What’s the weather in New York?”) or use voice input if configured.
- View the agent’s responses and debug events in the “Events” tab
#### 6. Stop the Server
- Press `Ctrl+c` in the terminal to stop the web server.


## Rferences

[Google Agent Development Kit](https://google.github.io/adk-docs/)