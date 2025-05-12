# EasyOpenChat Documentation

**Library Link**: [https://pypi.org/project/easyopenchat/](https://pypi.org/project/easyopenchat/)

## Introduction

**EasyOpenChat** is a Python library designed to simplify the creation of chatbots powered by OpenRouter's language model APIs. It provides a beginner-friendly interface for building chatbots while offering advanced features for customization and scalability. The library supports multiple interfaces (CLI, GUI, and web API), a plugin system, persistent memory, and streaming responses, making it suitable for both simple prototypes and production-grade applications.

* **Version**: 0.3.0
* **License**: MIT
* **Python Version**: >= 3.8
* **Dependencies**: `requests`, `jinja2`, `gradio`, `fastapi`, `uvicorn`, `rich`
* **Optional Dependencies**: `faiss-cpu`, `numpy` (for vector memory)

## Purpose

The primary goal of EasyOpenChat is to lower the barrier to entry for developers building chatbots. It abstracts complex API interactions with OpenRouter, provides built-in conversation management, and offers flexible interfaces for deployment.

## Installation

### Prerequisites

* Python >= 3.8
* An OpenRouter API key (obtainable from [OpenRouter](https://openrouter.ai))

### Install via pip

```bash
pip install easyopenchat
```

### Optional Dependencies (for Vector Memory)

```bash
pip install easyopenchat[vector]
```

### Manual Installation

```bash
git clone https://github.com/your-repo/easyopenchat
cd easyopenchat
pip install -r requirements.txt
```

Or manually install:

```bash
pip install requests jinja2 gradio fastapi uvicorn rich
```

## Usage

### Basic Example (CLI)

```python
from easyopenchat import EasyChatBot

api_key = "your-openrouter-api-key"
bot = EasyChatBot(api_key=api_key)
bot.run_cli()
```

### Advanced Example (GUI with Custom Prompt)
#### Default Model used here : "google/gemini-2.0-flash-exp:free" from openrouter (its a free model)

```python
from easyopenchat import EasyChatBot

api_key = "your-openrouter-api-key"
bot = EasyChatBot(
    api_key=api_key,
    model="google/gemini-2.0-flash-exp:free",
    system_prompt="code_assistant",
)
bot.run_gui()
```

### Web API

```bash
uvicorn easyopenchat.web:app --host 0.0.0.0 --port 8000
```

### Plugin and Template Examples

See earlier section for plugin and template code samples.

---

## Features

* **API Integration**: Seamless interaction with OpenRouter's chat completion API.
* **Conversation Management**: Persistent history with JSON storage and optional vector-based semantic search.
* **Streaming Responses**: Real-time response streaming for CLI, GUI, and web interfaces.
* **Plugin System**: Extensible command system (e.g., `!time`, `!calc`) for custom functionality.
* **Prompt Templating**: Dynamic system prompts using Jinja2 templates.
* **Multiple Interfaces**:

  * CLI: Rich text formatting with `rich`.
  * GUI: Gradio-based interface with configuration and chat history.
  * Web API: FastAPI endpoints for programmatic access.
* **Error Handling**: Robust retry logic, user-friendly error messages, and exception handling.
* **Customization**: Configurable model, system prompt, history size, and vector memory.

---

## Architecture

### Core Modules

1. **chatbot.py (EasyChatBot class)**

   * The main entry point for creating and interacting with a chatbot.
   * Manages initialization, user input processing, plugin execution, and response generation.
   * Supports streaming and non-streaming responses.
   * Integrates memory, plugins, and prompt templates.

2. **client.py (OpenRouterClient class)**

   * Handles communication with OpenRouter's API.
   * Supports both synchronous (full response) and streaming (Server-Sent Events) requests.
   * Implements retry logic with exponential backoff for robust API calls.
   * Parses SSE streams to extract content for streaming responses.

3. **memory.py (Memory class)**

   * Manages conversation history using a JSON-based persistent storage system.
   * Supports message pruning to limit history size.
   * Provides methods for adding, loading, saving, and resetting history.

4. **vector\_memory.py (VectorMemory class)**

   * Optional module for semantic search using FAISS (disabled by default).
   * Stores embeddings and metadata for advanced context retrieval (placeholder implementation).
   * Requires `faiss-cpu` and `numpy` dependencies.

5. **prompts.py (PromptTemplate class)**

   * Uses Jinja2 for dynamic prompt templating.
   * Supports loading templates from files or raw strings.
   * Enables customizable system prompts for different chatbot behaviors.

### Interface Modules

6. **cli.py**

   * Provides a command-line interface using the `rich` library for formatted output.
   * Supports streaming responses, plugin commands, and history reset.

7. **gui.py**

   * Implements a Gradio-based graphical interface.
   * Includes configuration UI for API key, model, and system prompt.
   * Supports streaming responses with incremental updates (requires Gradio >= 3.50.0).

8. **web.py**

   * Implements a FastAPI-based web API.
   * Provides endpoints for configuration (`/configure`), chatting (`/chat`), and history reset (`/reset`).
   * Supports streaming responses via Server-Sent Events.

### Plugin System

9. **plugins/plugin\_loader.py**

   * Dynamically loads plugins from the `plugins` directory.
   * Plugins are Python functions prefixed with `plugin_` in files ending with `_plugin.py`.

10. **plugins/time\_plugin.py and calc\_plugin.py**

    * Example plugins for retrieving the current time and evaluating mathematical expressions.
    * Users can add custom plugins by following the same structure.

### Templates

11. **templates/helpful\_assistant.j2 and code\_assistant.j2**

    * Pre-built Jinja2 templates for common chatbot roles.
    * Users can add custom templates in the `templates` directory.

---

## Development and Scaling

### Adding Plugins

1. Create a file in `easyopenchat/plugins` (e.g., `my_plugin.py`).
2. Define a function prefixed with `plugin_`.

### Adding Prompt Templates

1. Create a `.j2` file in `easyopenchat/templates`.
2. Use it via `system_prompt="your_template"`.

### Implementing Vector Memory

Update `vector_memory.py` with your embedding logic using `sentence-transformers`.

---

## Testing

```bash
pytest tests/
```

Example:

```python
def test_chatbot_init():
    bot = EasyChatBot(api_key="test-key", model="test-model")
    assert bot.client.api_key == "test-key"
```

---

## Deployment

### Build and Upload to PyPI

```bash
python -m build
twine upload dist/*
```

### Docker Deployment

```dockerfile
FROM python:3.8
WORKDIR /app
COPY . .
RUN pip install .
CMD ["uvicorn", "easyopenchat.web:app", "--host", "0.0.0.0", "--port", "8000"]
```

```bash
docker build -t easyopenchat .
docker run -p 8000:8000 easyopenchat
```

---

## Limitations

* Vector memory is currently a placeholder and needs real embedding logic.
* Gradio >= 3.50.0 required for GUI streaming.
* Model access limited to those available on OpenRouter.
* Performance may drop with large chat histories.

---

## Troubleshooting

* **Streaming Issues**: Upgrade Gradio, verify SSE implementation.
* **API Errors**: Check API key and model.
* **Plugin Errors**: Check plugin naming and directory placement.

---

## Future Improvements

* Full vector memory with real embedding models
* Broader LLM provider support
* More interactive and customizable GUI
* Advanced async plugins
* Performance improvements for large histories

---

## License

Distributed under the MIT License.

---

## Contact

**Author**: Sriram G
**Email**: [sriramkrish379@gmail.com](mailto:sriramkrish379@gmail.com)


