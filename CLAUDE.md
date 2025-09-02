# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pinokio is a 1-click launcher for any open-source project, built as an Electron application with a focus on script isolation and security. It provides a user-friendly interface that can programmatically interact with scripts while maintaining security through isolation.

## Architecture

### Core Components
- **Main Process**: `main.js` - Entry point that determines whether to run in full or minimal mode
- **Full Mode**: `full.js` - Complete desktop application with windowed interface
- **Minimal Mode**: `minimal.js` - System tray-only mode for lightweight operation
- **Core Engine**: Uses `pinokiod` package (v3.41.0) as the backend engine
- **Configuration**: `config.js` - Centralized configuration using electron-store

### Application Modes
The application can run in two modes determined by the stored preference:
- **Full Mode** (default): Complete windowed interface with browser-like functionality
- **Minimal Mode**: System tray only with browser integration

### Window Management
- Uses `electron-window-state` for persistent window positioning
- Custom title bar styling with theme support
- Protocol handler for `pinokio://` URLs
- Multi-window support with proper isolation

## Development Commands

### Basic Development
```bash
npm start              # Launch in development mode
npm install            # Install dependencies
npm run monkeypatch    # Apply required patches to dependencies
```

### Building and Distribution
```bash
npm run pack           # Build for current platform (development)
npm run dist           # Build Linux distribution packages
npm run dist2          # Build for multiple platforms (macOS, Windows, Linux)
npm run zip            # Create distribution archives
```

### Platform-Specific Builds
```bash
npm run l              # Linux build (using Docker)
npm run mw             # macOS and Windows build
npm run build2         # Combined Linux + macOS/Windows build
```

### Post-Install Setup
```bash
npm run postinstall2   # Apply monkeypatch and install native dependencies
npm run fix            # Install required system dependencies (macOS: fpm via brew)
```

## Key Technical Details

### Security Architecture
- Scripts run in isolated environment under `~/pinokio/api`
- Binaries installed to `~/pinokio/bin` 
- Virtual environment isolation using `venv` attribute
- Path-based execution constraints using `path` attribute
- Script verification process for "Discover" page featured scripts

### Electron Configuration
- Disabled web security for local content
- Custom preload script (`preload.js`)
- Protocol handling for deep linking
- Certificate error handling for local development
- CORS and CSP header manipulation for embedded content

### Dependencies
- **Runtime**: `pinokiod` (^3.41.0) - Core functionality
- **UI**: `electron` (^23.1.2) with custom window management
- **Build**: `electron-builder` (^26.0.18) with notarization support
- **Storage**: `electron-store` for persistent configuration

### Monkeypatching
The build process includes patching of dependencies:
- `temp/yarn.js` → `node_modules/app-builder-lib/out/util/yarn.js`
- `temp/rebuild.js` → `node_modules/@electron/rebuild/lib/src/rebuild.js`

This is required before running `electron-builder install-app-deps`.

## Script Execution Context
All Pinokio scripts use JSON syntax and run within controlled environments:
- Default execution path: each app's directory
- Virtual environment activation via `venv` attribute
- Path specification via `path` attribute
- Message display via `message` attribute

Example script structure:
```json
{
  "method": "shell.run",
  "params": {
    "message": "uv pip install -r requirements.txt",
    "path": "server",
    "venv": "venv"
  }
}
```