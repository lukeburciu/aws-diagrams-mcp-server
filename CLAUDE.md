# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development Setup
```bash
# Install development dependencies
uv pip install -e ".[dev]"

# Install Python 3.10+ using uv
uv python install 3.10
```

### Testing
```bash
# Run all tests with verbose output
pytest -xvs tests/

# Run specific test file
pytest -xvs tests/test_models.py

# Run with coverage report
pytest --cov=lukeburciu.aws_diagram_mcp_server --cov-report=term-missing tests/

# Run specific test class or method
pytest -xvs tests/test_models.py::TestDiagramType
pytest -xvs tests/test_models.py::TestDiagramType::test_diagram_type_values
```

### Code Quality
```bash
# Run linter (configured in pyproject.toml)
ruff check .
ruff format .

# Run type checker
pyright

# Run security scanner
bandit -c pyproject.toml -r lukeburciu/
```

### Building
```bash
# Build Docker image
docker build -t lukeburciu/aws-diagram-mcp-server .

# Run Docker container
docker run --rm --interactive --env FASTMCP_LOG_LEVEL=ERROR lukeburciu/aws-diagram-mcp-server:latest
```

## Architecture

This is an MCP (Model Context Protocol) server that generates diagrams using the Python `diagrams` package. The server architecture follows:

### Core Components

1. **MCP Server (`server.py`)**: FastMCP-based server exposing four main tools:
   - `generate_diagram`: Creates PNG diagrams from Python code
   - `generate_diagram_dot`: Creates Graphviz DOT files from Python code
   - `list_diagram_icons`: Lists available diagram icons organized by provider
   - `get_diagram_examples`: Provides example code for different diagram types

2. **Security Scanner (`scanner.py`)**: 
   - Uses Bandit to scan diagram code for security issues before execution
   - Validates that only safe diagram-related imports are used
   - Prevents execution of potentially dangerous code

3. **Diagram Tools (`diagrams_tools.py`)**:
   - Handles actual diagram generation using the `diagrams` package
   - Manages workspace directory for generated diagrams
   - Provides icon discovery and example generation

4. **Models (`models.py`)**: 
   - Defines `DiagramType` enum for supported diagram types (aws, sequence, flow, class, k8s, onprem, custom)
   - Data validation using Pydantic

### Key Design Patterns

- **Security-First**: All user-provided Python code is scanned with Bandit before execution
- **Sandboxed Execution**: Diagram code runs with restricted imports and no file system access beyond the workspace
- **FastMCP Framework**: Leverages FastMCP for simplified MCP server implementation
- **Async Support**: Server tools support async operations for better performance

### Supported Diagram Types

The server supports multiple diagram types through the `diagrams` Python package:
- AWS Architecture Diagrams
- Kubernetes Diagrams  
- On-Premise Infrastructure
- Sequence Diagrams
- Flow Charts
- Class Diagrams
- Custom Diagrams

### Testing Strategy

Tests are organized by module with fixtures in `conftest.py`:
- Unit tests for models, scanner, and diagram generation
- Integration tests for MCP server tools
- Mock external dependencies (file system, diagram generation)
- Async test support with pytest-asyncio