# Claude Xcode Plugin

Claude Code plugin that bundles Apple's official Xcode MCP server with best-practice skills for SwiftUI macOS projects.

Installing this plugin automatically connects Claude Code to Xcode — no manual `claude mcp add` needed.

## Prerequisites

1. **Xcode** must be installed (26.0+)

2. **xcode-select** must point to your Xcode installation:
   ```
   sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
   ```

3. **Enable MCP in Xcode settings:**
   - Open Xcode > Settings > Intelligence
   - Under Model Context Protocol, toggle **Xcode Tools** on

## Installation

```
/plugin marketplace add daig/dai-claude-marketplace
/plugin install xcode-mcp@dai-claude-marketplace
```

The Xcode MCP server (`xcrun mcpbridge`) starts automatically when the plugin is enabled.

## What's Included

### MCP Server

The plugin bundles Apple's Xcode MCP server, providing 20 tools for building, testing, previewing, searching, and editing Xcode projects directly from Claude Code.

### Skill: xcode-mcp

Best practices learned from real-world usage of the Xcode MCP tools:

- **When to use Xcode MCP vs filesystem tools** — decision table for every operation
- **XcodeWrite file placement pitfall** — known issue where new files land at project root
- **Filesystem-synced groups** — how modern Xcode projects auto-compile files
- **Adding test targets** — complete pbxproj editing guide for unit test and UI test targets
- **SwiftUI + XCUITest on macOS** — accessibility patterns that actually work (button styles, accessibilityValue, waitForExistence)
- **Build/test workflow** — step-by-step using MCP tools

## Example Prompts

- "Add unit tests to this Xcode project"
- "Why can't XCUITest find my SwiftUI buttons?"
- "Set up a UI test target for my macOS app"
- "Build and run tests using the Xcode MCP"

## License

MIT
