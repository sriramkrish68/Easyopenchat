Library Link: https://pypi.org/project/easyopenchat/

# EasyOpenChat Documentation

## Overview

**EasyOpenChat** is a Python library designed to simplify the creation of chatbots powered by OpenRouter's language model APIs. It provides a beginner-friendly interface for building chatbots while offering advanced features for customization and scalability. The library supports multiple interfaces (CLI, GUI, and web API), a plugin system, persistent memory, and streaming responses, making it suitable for both simple prototypes and production-grade applications.

- **Version**: 0.3.0
- **License**: MIT
- **Python Version**: >= 3.8
- **Dependencies**: requests, jinja2, gradio, fastapi, uvicorn, rich
- **Optional Dependencies**: faiss-cpu, numpy (for vector memory)

## Purpose

The primary goal of EasyOpenChat is to lower the barrier to entry for developers building chatbots. It abstracts complex API interactions with OpenRouter, provides built-in conversation management, and offers flexible interfaces for deployment. Key features include:

- **Ease of Use**: Minimal code required to create a functional chatbot.
- **Extensibility**: Plugin system and prompt templating for custom functionality.
- **Multiple Interfaces**: CLI, Gradio-based GUI, and FastAPI-based web API.
- **Robustness**: Error handling, retries, and persistent memory.
- **Streaming Support**: Real-time response streaming for interactive experiences.

## Architecture

The library is modular, with distinct components handling specific functionalities. Below is an overview of the key modules and their roles:

### Core Modules

1. **chatbot.py (EasyChatBot class)**:
  
  - The main entry point for creating and interacting with a chatbot.
  - Manages initialization, user input processing, plugin execution, and response generation.
  - Supports streaming and non-streaming responses.
  - Integrates memory, plugins, and prompt templates.
2. **client.py (OpenRouterClient class)**:
  
  - Handles communication with OpenRouter's API.
  - Supports both synchronous (full response) and streaming (Server-Sent Events) requests.
  - Implements retry logic with exponential backoff for robust API calls.
  - Parses SSE streams to extract content for streaming responses.
3. **memory.py (Memory class)**:
  
  - Manages conversation history using a JSON-based persistent storage system.
  - Supports message pruning to limit history size.
  - Provides methods for adding, loading, saving, and resetting history.
4. **vector_memory.py (VectorMemory class)**:
  
  - Optional module for semantic search using FAISS (disabled by default).
  - Stores embeddings and metadata for advanced context retrieval (placeholder implementation).
  - Requires `faiss-cpu` and `numpy` dependencies.
5. **prompts.py (PromptTemplate class)**:
  
  - Uses Jinja2 for dynamic prompt templating.
  - Supports loading templates from files or raw strings.
  - Enables customizable system prompts for different chatbot behaviors.

### Interface Modules

6. **cli.py**:
  
  - Provides a command-line interface using the `rich` library for formatted output.
  - Supports streaming responses, plugin commands, and history reset.
7. **gui.py**:
  
  - Implements a Gradio-based graphical interface.
  - Includes configuration UI for API key, model, and system prompt.
  - Supports streaming responses with incremental updates (requires Gradio >= 3.50.0).
8. **web.py**:
  
  - Implements a FastAPI-based web API.
  - Provides endpoints for configuration (`/configure`), chatting (`/chat`), and history reset (`/reset`).
  - Supports streaming responses via Server-Sent Events.

### Plugin System

9. **plugins/plugin_loader.py**:
  
  - Dynamically loads plugins from the `plugins` directory.
  - Plugins are Python functions prefixed with `plugin_` in files ending with `_plugin.py`.
10. **plugins/time_plugin.py and calc_plugin.py**:
  
  - Example plugins for retrieving the current time and evaluating mathematical expressions.
  - Users can add custom plugins by following the same structure.

### Templates

11. **templates/helpful_assistant.j2 and code_assistant.j2**:
  - Pre-built Jinja2 templates for common chatbot roles (helpful assistant and code assistant).
  - Users can add custom templates in the `templates` directory.

## Features

- **API Integration**: Seamless interaction with OpenRouter's chat completion API.
- **Conversation Management**: Persistent history with JSON storage and optional vector-based semantic search.
- **Streaming Responses**: Real-time response streaming for CLI, GUI, and web interfaces.
- **Plugin System**: Extensible command system (e.g., `!time`, `!calc`) for custom functionality.
- **Prompt Templating**: Dynamic system prompts using Jinja2 templates.
- **Multiple Interfaces**:
  - CLI: Rich text formatting with `rich`.
  - GUI: Gradio-based interface with configuration and chat history.
  - Web API: FastAPI endpoints for programmatic access.
- **Error Handling**: Robust retry logic, user-friendly error messages, and exception handling.
- **Customization**: Configurable model, system prompt, history size, and vector memory.

## Installation

### Prerequisites

- Python >= 3.8
- An OpenRouter API key (obtainable from [OpenRouter](https://openrouter.ai)).

### Install via pip

```bash
pip install easyopenchat
```

### Optional Dependencies (for Vector Memory)

```bash
pip install easyopenchat[vector]
```

### Manual Installation

1. Clone the repository or download the source code.
2. Navigate to the project directory:
  
  ```bash
  cd easyopenchat
  ```
  
3. Install dependencies:
  
  ```bash
  pip install -r requirements.txt
  ```
  
  Or manually install:
  
  ```bash
  pip install requests jinja2 gradio fastapi uvicorn rich
  ```
  

## Usage

### Basic Example (CLI)

Create a simple chatbot with the CLI interface:

Default Model: google/gemini-2.0-flash-exp:free  (from openrouter)
```python
from easyopenchat import EasyChatBot

api_key = "your-openrouter-api-key"
bot = EasyChatBot(api_key=api_key)
bot.run_cli()
```

- **Interaction**:
  - Type messages to chat.
  - Use `!reset` to clear history.
  - Use `!time` or `!calc <expression>` for plugin commands.
  - Type `exit` to quit.

### Advanced Example (GUI with Custom Prompt)

Create a chatbot with a custom prompt and vector memory:

```python
from easyopenchat import EasyChatBot

api_key = "your-openrouter-api-key"
bot = EasyChatBot(
    api_key=api_key,
    model="openai/gpt-4",
    system_prompt="code_assistant",
    use_vector_memory=True,
    max_history=200
)
bot.run_gui()
```

- **GUI Features**:
  - Configure API key, model, and system prompt via the UI.
  - Chat with streaming responses.
  - Reset history with a button.

### Web API

Run the FastAPI server:

```bash
uvicorn easyopenchat.web:app --host 0.0.0.0 --port 8000
```

- **Endpoints**:
  - `POST /configure`: Configure the bot.
    
    ```bash
    curl -X POST "http://localhost:8000/configure" -H "Content-Type: application/json" -d '{"api_key": "your-api-key", "model": "openai/gpt-3.5-turbo", "prompt": "You are a helpful AI."}'
    ```
    
  - `POST /chat`: Send a message and get a response.
    
    ```bash
    curl -X POST "http://localhost:8000/chat" -H "Content-Type: application/json" -d '{"message": "Hello!", "stream": false}'
    ```
    
    For streaming:
    
    ```bash
    curl -X POST "http://localhost:8000/chat" -H "Content-Type: application/json" -d '{"message": "Hello!", "stream": true}'
    ```
    
  - `POST /reset`: Reset conversation history.
    
    ```bash
    curl -X POST "http://localhost:8000/reset"
    ```
    

### Plugin Example

Add a custom plugin to the `easyopenchat/plugins` directory:

```python
# plugins/greet_plugin.py
def plugin_greet(args):
    return f"Hello, {args or 'User'}!"
```

- **Usage**: In the CLI or GUI, type `!greet Alice` to get `Hello, Alice!`.

### Prompt Template Example

Create a custom prompt template in `easyopenchat/templates`:

```plaintext
# templates/custom_assistant.j2
You are a {{ role }} AI assistant. Respond in a {{ tone }} tone.
```

Use it:

```python
bot = EasyChatBot(api_key="your-api-key", system_prompt="custom_assistant")
bot.prompt_template.render(role="friendly", tone="casual")
```

## Configuration Options

The `EasyChatBot` class supports the following parameters:

- `api_key` (str): OpenRouter API key (required).
- `model` (str): Model name (default: `openai/gpt-3.5-turbo`).
- `system_prompt` (str): Custom prompt or template name (default: "You are a helpful AI assistant.").
- `use_vector_memory` (bool): Enable FAISS-based vector memory (default: False).
- `max_history` (int): Maximum number of messages to store (default: 100).

## Technical Details

### API Communication

- **Endpoint**: `https://openrouter.ai/api/v1/chat/completions`
- **Headers**:
  - `Authorization`: Bearer token with the API key.
  - `HTTP-Referer`: Set to `http://localhost`.
  - `X-Title`: Set to `easyopenchat`.
- **Streaming**: Uses Server-Sent Events (SSE) with `data:` prefixed JSON chunks.
- **Retry Logic**: Up to 3 retries with exponential backoff (2^attempt seconds).

### Memory Management

- **Storage**: JSON file (`chat_history.json` by default).
- **Format**: List of dictionaries with `role`, `content`, and `timestamp`.
- **Pruning**: Automatically limits history to `max_history` messages.
- **Vector Memory**: Placeholder implementation using FAISS; requires custom embedding integration.

### Streaming Responses

- **CLI**: Uses `rich` for real-time text output.
- **GUI**: Incremental history updates with Gradio (requires `queue=True` for Gradio >= 3.50.0).
- **Web API**: Returns NDJSON chunks (`{"chunk": "text"}`) for streaming requests.

### Error Handling

- **API Errors**: Retries on `requests.RequestException` with exponential backoff.
- **GUI Errors**: Displays errors in the chat history.
- **Plugin Errors**: Returns error messages for invalid commands or execution failures.


## Limitations

- **Vector Memory**: Placeholder implementation; requires custom embedding integration for production use.
- **GUI Streaming**: Requires Gradio >= 3.50.0 for streaming support due to queue requirements.
- **Model Support**: Limited to models available via OpenRouter.
- **Performance**: Large conversation histories may impact performance without optimization.

## Troubleshooting

- **GUI Error: "Need to enable queue to use generators"**:
  - Upgrade Gradio to >= 3.50.0 (`pip install --upgrade gradio`).
  - Alternatively, use a non-streaming `gui.py` (see previous responses).
- **API Errors**: Verify your OpenRouter API key and model name. Check network connectivity.
- **Plugin Issues**: Ensure plugin files are in `easyopenchat/plugins` and follow the naming convention.
- **Streaming Issues**: Ensure `client.py` correctly parses SSE chunks (see previous fixes).

## Development and Scaling Further


## Extending the Library

### Adding Plugins

1. Create a file in `easyopenchat/plugins` (e.g., `my_plugin.py`).
2. Define a function prefixed with `plugin_`:
  
  ```python
  def plugin_mycommand(args):
      return f"Custom command with args: {args}"
  ```
  
3. Use the plugin with `!mycommand` in any interface.

### Adding Prompt Templates

1. Create a `.j2` file in `easyopenchat/templates` (e.g., `my_template.j2`):
  
  ```plaintext
  You are a {{ role }} AI. Use a {{ tone }} tone.
  ```
  
2. Initialize the bot with the template:
  
  ```python
  bot = EasyChatBot(api_key="your-api-key", system_prompt="my_template")
  ```
  

### Implementing Vector Memory

The current `vector_memory.py` uses a placeholder implementation. To enable semantic search:

1. Install dependencies:
  
  ```bash
  pip install faiss-cpu numpy sentence-transformers
  ```
  
2. Update `vector_memory.py` to generate embeddings:
  
  ```python
  from sentence_transformers import SentenceTransformer
  import faiss
  import numpy as np
  
  class VectorMemory:
      def __init__(self, dimension=384):
          self.model = SentenceTransformer('all-MiniLM-L6-v2')
          self.dimension = dimension
          self.index = faiss.IndexFlatL2(dimension)
          self.metadata = []
  
      def add(self, text, metadata):
          embedding = self.model.encode(text, convert_to_numpy=True)
          self.index.add(embedding.reshape(1, -1))
          self.metadata.append(metadata)
  
      def search(self, query_text, k=5):
          query_embedding = self.model.encode(query_text, convert_to_numpy=True)
          distances, indices = self.index.search(query_embedding.reshape(1, -1), k)
          return [(distances[0][i], self.metadata[indices[0][i]]) for i in range(len(indices[0]))]
  ```
  
3. Enable vector memory:
  
  ```python
  bot = EasyChatBot(api_key="your-api-key", use_vector_memory=True)
  ```
  

## Testing

The library includes a basic test suite in `tests/test_chatbot.py`. Run tests with:

```bash
pytest tests/
```

Example test:

```python
def test_chatbot_init():
    bot = EasyChatBot(api_key="test-key", model="test-model")
    assert bot.client.api_key == "test-key"
    assert bot.client.model == "test-model"
    assert len(bot.memory.history) == 1  # System prompt
```

## Deployment

### Packaging

Build a distributable package:

```bash
python -m build
```

Upload to PyPI:

```bash
twine upload dist/*
```

### Running the Web API

Deploy the FastAPI server with a production-grade setup:

```bash
uvicorn easyopenchat.web:app --host 0.0.0.0 --port 8000 --workers 4
```

### Docker Deployment

Create a `Dockerfile`:

```dockerfile
FROM python:3.8
WORKDIR /app
COPY . .
RUN pip install .
CMD ["uvicorn", "easyopenchat.web:app", "--host", "0.0.0.0", "--port", "8000"]
```

Build and run:

```bash
docker build -t easyopenchat .
docker run -p 8000:8000 easyopenchat
```

## Future Improvements

- Full vector memory implementation with embedding models.
- Support for additional LLM providers beyond OpenRouter.
- Enhanced GUI features (e.g., chat history export, theme customization).
- Advanced plugin system with async support and plugin configuration.
- Performance optimizations for large conversation histories.

## License

Distributed under the MIT License. See `LICENSE` file for details.

## Contact

- **Author**: Sriram G (sriramkrish379@gmail.com)
- **Repository**: [GitHub placeholder]
