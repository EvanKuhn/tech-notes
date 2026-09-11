## New Tools, 2026
### Command-line tools
- `uv` 
	- Fast replacement for `pip`, `pip-tools`, `virtualenv`, etc
	- Super fast, built with Rust.
- `ruff`
	- An extremely fast Python linter and code formatter
	- Use instead of `black` ?
- `ty`
	- An extremely fast Python type checker.
	- Use instead of `mypy`, which sucks
### Web serving
- `fastapi`
	- A modern, high-performance web framework used for building APIs with Python. 
	- Very popular for backend development, especially for microservices, real-time applications, and artificial intelligence (AI) backends.
	- Uses event-driven async IO
	- Automatically builds documentation
	- Relies heavily on type hints for run-time error-checking
	- Built upon existing packages:
		- `Starlette`: Handles the web routing, connection management, and asynchronous capabilities.
		- `Pydantic`: Handles data validation, serialization, and schema definition.
		- `Uvicorn`: Lightning-fast ASGI web server.


