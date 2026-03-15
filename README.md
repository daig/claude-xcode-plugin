# Claude Xcode Plugin

Claude Code skill plugin for working with Xcode MCP tools and SwiftUI macOS projects.

## Installation

```
/plugin marketplace add daig/dai-claude-marketplace
/plugin install xcode-mcp@dai-claude-marketplace
```

## Skills

| Skill | Description |
|-------|-------------|
| **xcode-mcp** | Xcode MCP tool best practices, file operation tradeoffs, test target setup, XCUITest patterns |

## What's Covered

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
