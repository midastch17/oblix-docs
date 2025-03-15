# Oblix SDK Documentation

[![GitHub stars](https://img.shields.io/github/stars/oblix-ai/docs?style=social)](https://github.com/oblix-ai/docs/stargazers)
[![License](https://img.shields.io/github/license/oblix-ai/docs)](https://github.com/oblix-ai/docs/blob/main/LICENSE)

Official documentation for the Oblix SDK - Intelligent AI model orchestration between local and cloud models.

## About Oblix

Oblix SDK provides:
- **Intelligent AI Orchestration**: Seamlessly orchestrate between edge and cloud models
- **Seamless Model Switching**: Automatically switch between local edge models and cloud APIs
- **Connectivity Awareness**: Gracefully handle offline scenarios with intelligent fallbacks
- **Resource Optimization**: Dynamically orchestrate based on system resources

## Documentation Website

Visit our documentation at: [https://documentation.oblix.ai](https://documentation.oblix.ai)

## Contributing

We welcome contributions to improve our documentation! Here's how you can help:

### Prerequisites

- Python 3.8 or higher
- pip (Python package installer)

### Setup

1. Fork the repository on GitHub
2. Clone your fork locally:
   ```bash
   git clone https://github.com/YOUR-USERNAME/docs.git
   cd docs
   ```
3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

### Local Development

1. Start the documentation server:
   ```bash
   mkdocs serve
   ```
2. Visit `http://localhost:8000` to preview your changes

### Making Changes

1. Create a new branch for your changes:
   ```bash
   git checkout -b feature/your-feature-name
   ```
2. Make your changes to the Markdown files in the `docs/` directory
3. Preview your changes locally with `mkdocs serve`
4. Commit your changes:
   ```bash
   git commit -am "Add your descriptive commit message"
   ```
5. Push to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
6. Submit a pull request from your fork to the main repository

### Pull Request Guidelines

- Ensure your PR includes a clear description of the changes
- Link any related issues
- Update or add documentation as needed
- Follow existing style and formatting

## Project Structure

- `docs/`: Documentation source files (Markdown)
- `mkdocs.yml`: MkDocs configuration file
- `requirements.txt`: Python dependencies
- `site/`: Generated static site (don't edit directly)

## License

This project is licensed under the [MIT License](LICENSE).

## Contact

- GitHub: [oblix-ai/docs](https://github.com/oblix-ai/docs)
- Twitter: [@oblixai](https://twitter.com/oblixai)
- LinkedIn: [oblixai](https://linkedin.com/company/oblixai)