# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GitHub Copilot for Xcode is a macOS application that provides AI-powered code completion and chat assistance for Xcode. It integrates GitHub Copilot's capabilities directly into the Xcode development environment through Xcode source editor extensions.

## Build Commands

### Local Development Build
```bash
cd ./Script
sh ./uninstall-app.sh    # Remove any previous installation
del ../build             # Clean the build directory (use del instead of rm)
sh ./localbuild-app.sh   # Build a fresh copy of the app
```

### Server Components (Node.js/TypeScript)
```bash
cd Server
npm install              # Install dependencies
npm run build           # Build with webpack
```

### Testing
```bash
# Run all unit tests (from Xcode or command line)
xcodebuild test -scheme "Copilot for Xcode" -workspace "Copilot for Xcode.xcworkspace"

# Tests are configured in TestPlan.xctestplan
# All new tests should be added to this test plan
```

### Code Formatting
- Uses SwiftFormat for Swift code formatting
- Follows Ray Wenderlich Style Guide with 4 spaces for indentation

## Architecture Overview

The project is organized into several key targets and packages:

### Main Targets
- **Copilot for Xcode**: Host app containing XPCService and editor extension, provides settings UI
- **EditorExtension**: Xcode source editor extension that forwards editor content to XPCService
- **ExtensionService**: Background service where all core features are implemented
- **CommunicationBridge**: Maintains communication between host app/editor extension and ExtensionService

### Swift Packages
- **Core**: Contains main application logic organized by feature areas
  - `Service`: ExtensionService implementation
  - `HostApp`: Host application implementation
  - `SuggestionWidget`: UI components for code suggestions
  - `ConversationTab`: Chat interface components
  - `ChatService`: Chat functionality and context management
  - `SuggestionService`: Code completion service
  - `PromptToCodeService`: Code generation from prompts

- **Tool**: Shared utilities and lower-level services
  - `GitHubCopilotService`: Core integration with GitHub Copilot Language Server
  - `Workspace`: File system and project management
  - `XcodeInspector`: Xcode app monitoring and interaction
  - `SuggestionBasic`: Core suggestion data types
  - `Preferences`: Configuration management
  - `Logger`: Logging infrastructure

- **Server**: Node.js/TypeScript components
  - Monaco editor integration for diff views
  - Terminal/xterm integration
  - Webpack-based build system

### Key Service Architecture
- Uses actor-based concurrency with `@WorkspaceActor` and `@ServiceActor`
- Dependency injection via swift-dependencies
- Composable Architecture (TCA) for UI state management
- XPC communication between sandboxed and non-sandboxed components

### Data Flow
1. Editor extension captures Xcode content via source editor APIs
2. Content forwarded to ExtensionService via CommunicationBridge XPC
3. ExtensionService processes requests through GitHubCopilotService 
4. Language Server Protocol (LSP) communication with GitHub Copilot backend
5. Results returned through same XPC chain back to editor

## Prerequisites

- macOS 12+
- Xcode 8+
- Node.js and npm (symlinked to /usr/local/bin for Xcode run scripts)
- GitHub Copilot subscription

## Key Files to Understand

- `Core/Sources/Service/Service.swift`: Main service entry point
- `Tool/Sources/GitHubCopilotService/LanguageServer/GitHubCopilotService.swift`: Core Copilot integration
- `Core/Package.swift` & `Tool/Package.swift`: Swift package configurations
- `TestPlan.xctestplan`: Centralized test configuration
- `Version.xcconfig`: Version control for all targets
- `DEVELOPMENT.md`: Detailed development setup and architecture notes

## Common Development Patterns

- All async operations use Swift concurrency (async/await)
- UI built with SwiftUI using Composable Architecture patterns  
- Preference storage via UserDefaults with type-safe property wrappers
- Logging via centralized Logger package with different log levels
- File watching and workspace management through dedicated services
- XPC communication follows request-response patterns with proper error handling