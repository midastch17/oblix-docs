# Installation Guide

This guide walks you through the process of installing Oblix on your macOS system.

## System Requirements

- **Operating System**: macOS 11 (Big Sur) or newer
- **Storage**: 800 MB of free disk space
- **Memory**: 8 GB RAM recommended
- **Internet Connection**: Required for cloud model access

## Installation Steps

1. **Download the Installer**: 
   - Visit [Oblix AI](https://oblix.ai)
   - Download the latest Oblix installer (.dmg file)

2. **Open the Installer**:
   - Locate the downloaded .dmg file in your Downloads folder
   - Double-click the .dmg file to mount it
   - A new Finder window will open showing the Oblix application and Applications folder

3. **Install Oblix**:
   - Drag the Oblix application icon to the Applications folder
   - Close the installer window

4. **Run Oblix for the First Time**:
   - Navigate to your Applications folder
   - Double-click the Oblix application
   - macOS may display a security warning the first time you open it
   - Click "Open" if prompted

5. **Complete Installation**:
   - You'll be prompted to enter your system password
   - This is required to install necessary components
   - After entering your password, the installation will complete
   - You should see a confirmation message that Oblix has been successfully installed

6. **Verify Installation**:
   - Open Terminal
   - Run the following command to verify Oblix is properly installed:
     ```bash
     oblix --version
     ```
   - You should see the current version of Oblix displayed

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

**Issue**: "Oblix can't be opened because it is from an unidentified developer"
**Solution**: Go to System Preferences > Security & Privacy > General, and click "Open Anyway"

**Issue**: Command not found when trying to use Oblix in Terminal
**Solution**: Make sure the installation completed successfully. You might need to restart your Terminal or add Oblix to your PATH.

**Issue**: System password prompt appears multiple times
**Solution**: This is normal behavior if you cancel the password prompt. Enter your password to continue installation.

### Getting Help

If you encounter issues during installation:

1. Join our Discord community: [https://discord.gg/v8qtEVuU](https://discord.gg/v8qtEVuU)
2. Contact support at [team@oblix.ai](mailto:team@oblix.ai)

## Uninstallation

To uninstall Oblix:

1. Open Applications folder in Finder
2. Drag Oblix to the Trash
3. Empty the Trash
4. Optional: Remove configuration files with:
   ```bash
   rm -rf ~/.oblix
   ```