# Installation Guide

This guide walks you through the process of installing Oblix on your macOS system.

## System Requirements

- **Operating System**: macOS 11 (Big Sur) or newer
- **Python Versions**: 3.9, 3.10, 3.11, 3.12, 3.13
- **Storage**: 800 MB of free disk space
- **Memory**: 8 GB RAM recommended
- **Internet Connection**: Required for cloud model access

## Installation with pip

The recommended way to install Oblix is using pip:

```bash
pip install oblix
```

For a specific Python version (if you have multiple Python versions installed):

```bash
python3.11 -m pip install oblix
```

### Verify Installation

After installation, verify that Oblix is properly installed:

```bash
oblix --version
```

You should see the current version of Oblix displayed.

## Using Oblix

After installation, you'll need to set up environment variables before using Oblix from the Terminal.

### Required Environment Variables

For Oblix to function properly, you must set the following environment variables:

1. **OBLIX_API_KEY** - Your Oblix API key
2. **OPENAI_API_KEY** - Your OpenAI API key (for using GPT models)
3. **ANTHROPIC_API_KEY** - Your Anthropic API key (for using Claude models)

You can set these in your terminal session or add them to your shell profile for persistence:

```bash
# Add these lines to your ~/.zshrc or ~/.bash_profile
export OBLIX_API_KEY="your_oblix_api_key_here"
export OPENAI_API_KEY="your_openai_api_key_here"
export ANTHROPIC_API_KEY="your_anthropic_api_key_here"
```

After adding these to your profile file, apply the changes:

```bash
source ~/.zshrc  # or source ~/.bash_profile
```

### Using the CLI

Once environment variables are set, you can use Oblix from the Terminal:

```bash
# Get help with available commands
oblix --help

# Start an interactive chat session
oblix chat --local-model llama2 --cloud-model gpt-3.5-turbo
```

You can also provide API keys directly in commands:

```bash
oblix chat --local-model llama2 --cloud-model gpt-3.5-turbo --cloud-api-key "your_api_key_here"
```

### Virtual Environment (Recommended)

It's recommended to install Oblix in a virtual environment:

```bash
# Create and activate virtual environment
python -m venv oblix-env
source oblix-env/bin/activate  # On Windows: oblix-env\Scripts\activate

# Install Oblix
pip install oblix
```

## Installing Ollama (Required for Local Models)

To use local models with Oblix, you'll need to install Ollama:

1. Download Ollama from [ollama.ai](https://ollama.ai)
2. Follow the installation instructions for macOS
3. Verify Ollama is running by opening Terminal and typing:
   ```bash
   ollama list
   ```

## Troubleshooting

### Common Issues

**Issue**: Command not found when trying to use Oblix in Terminal
**Solution**: Make sure the installation completed successfully. You might need to restart your Terminal or add the Python scripts directory to your PATH.

**Issue**: Permission error during installation
**Solution**: Try installing with `pip install --user oblix` or use a virtual environment.

**Issue**: Import errors after installation
**Solution**: Ensure you're using a supported Python version (3.9, 3.10, 3.11, 3.12, or 3.13).

### Getting Help

If you encounter issues during installation:

1. Join our Discord community: [https://discord.gg/v8qtEVuU](https://discord.gg/v8qtEVuU)
2. Contact support at [team@oblix.ai](mailto:team@oblix.ai)

## Uninstallation

To uninstall Oblix:

### Pip Installation

If you installed Oblix using pip:

```bash
pip uninstall oblix
```

### Configuration Files

To remove configuration files (optional):

```bash
rm -rf ~/.oblix
```