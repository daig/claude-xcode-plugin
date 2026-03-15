---
name: xcode-mcp
description: >
  Best practices for using Xcode MCP server tools with SwiftUI macOS projects.
  Covers when to use Xcode MCP tools vs filesystem tools, file placement pitfalls,
  adding test targets to pbxproj, XCUITest accessibility patterns on macOS, and
  build/test workflows. Use when working with Xcode projects, SwiftUI macOS apps,
  XCUITest UI tests, or the Xcode MCP server.
---

# Xcode MCP — Practical Reference

## Quick Reference: Which Tool Should I Use?

| Operation | Use | Reason |
|-----------|-----|--------|
| Read/edit existing file contents | `Read`, `Edit`, `Grep`, `Glob` | Faster, no `tabIdentifier` overhead, no Xcode dependency |
| Search across codebase | `Grep`, `Glob` | Richer options, works on all files including those outside the project |
| Create new file in synced group | Filesystem `Write` + `mkdir` | Auto-compiled by Xcode; avoids XcodeWrite placement bugs |
| Create new file in non-synced project | `XcodeWrite` | Adds to project graph automatically |
| Delete/move/rename a file | `XcodeRM`, `XcodeMV` | Updates project references; filesystem-only ops leave dangling refs |
| Create directory/group | `XcodeMakeDir` or filesystem `mkdir` | Both work for synced groups; use MCP for non-synced |
| Check what Xcode sees vs filesystem | `XcodeLS`, `XcodeGlob` | Detects stale refs or excluded files |
| Live compiler diagnostics | `XcodeRefreshCodeIssuesInFile` | Only available through Xcode |
| Build project | `BuildProject` | Only available through Xcode |
| Run tests | `RunAllTests`, `RunSomeTests` | Only available through Xcode |
| Launch app for manual testing | User presses Cmd+R in Xcode | No MCP tool exists for this |

## Critical: XcodeWrite File Placement

`XcodeWrite` for **new** files may place them at the project root instead of inside the target source directory, even when the path looks correct (e.g., `my-app/NewFile.swift` may land at `<project_root>/NewFile.swift` instead of `<project_root>/my-app/NewFile.swift`).

**Always:**
1. Check the `absolutePath` field in the XcodeWrite response
2. If wrong, move the file to the correct location with `mv`
3. Use `XcodeRM` to remove the stale project reference at the wrong path
4. Prefer filesystem `Write` for projects using `PBXFileSystemSynchronizedRootGroup`

## Filesystem-Synced vs Explicit File Lists

Modern Xcode projects (objectVersion 77+) use `PBXFileSystemSynchronizedRootGroup`. Any `.swift` file placed in a synced directory is automatically compiled into that target. No project graph mutation is needed for adding files.

Check if a project uses synced groups:
```
# In project.pbxproj, look for:
isa = PBXFileSystemSynchronizedRootGroup;
```

If present: create files directly on the filesystem. They are auto-compiled.
If absent: use `XcodeWrite` to ensure files are added to the target.

## Session Setup

Every Xcode MCP tool call requires a `tabIdentifier`. Fetch it once at session start:

```
XcodeListWindows → tabIdentifier: "windowtab1"
```

Cache this value for all subsequent calls. It only changes if the user opens a different workspace.

## Adding Test Targets to pbxproj

Template Xcode projects often have no test targets. Add them by rewriting `project.pbxproj` directly (not via Xcode MCP — the MCP cannot create native targets).

**Required sections for each test target:**

| Section | What to add |
|---------|-------------|
| `PBXContainerItemProxy` | Proxy linking test target → app target |
| `PBXFileReference` | `.xctest` product reference |
| `PBXFileSystemSynchronizedRootGroup` | Synced directory for test sources |
| `PBXFrameworksBuildPhase` | Empty frameworks phase |
| `PBXGroup` | Add synced group to main group children; add .xctest to Products |
| `PBXNativeTarget` | Target definition with build phases, dependencies, product type |
| `PBXProject` | Add to `targets` array and `TargetAttributes` |
| `PBXResourcesBuildPhase` | Empty resources phase |
| `PBXSourcesBuildPhase` | Empty sources phase (synced group handles files) |
| `PBXTargetDependency` | Dependency on app target |
| `XCBuildConfiguration` | Debug + Release configs with target-specific settings |
| `XCConfigurationList` | Config list referencing the two configs |

**Unit test target build settings:**
```
BUNDLE_LOADER = "$(TEST_HOST)";
TEST_HOST = "$(BUILT_PRODUCTS_DIR)/MyApp.app/Contents/MacOS/MyApp";
PRODUCT_BUNDLE_IDENTIFIER = "com.example.MyAppTests";
productType = "com.apple.product-type.bundle.unit-test";
```

**UI test target build settings:**
```
TEST_TARGET_NAME = "MyApp";
PRODUCT_BUNDLE_IDENTIFIER = "com.example.MyAppUITests";
productType = "com.apple.product-type.bundle.ui-testing";
```

**ID strategy:** Use deterministic hex prefixes (e.g., `AAA0...` for unit tests, `BBB0...` for UI tests) that don't collide with existing IDs.

**Rewrite the full file** rather than incremental edits to avoid pbxproj corruption.

**Module naming:** `@testable import` uses underscores for hyphens: `my-app` → `my_app`.

## SwiftUI + XCUITest on macOS

### Buttons Must Have Explicit Style

Default SwiftUI `Button` on macOS is **invisible to XCUITest**. Always apply an explicit button style:

```swift
Button("1") { action() }
    .buttonStyle(.bordered)          // required for XCUITest visibility
    .accessibilityIdentifier("1")    // for test queries
```

### Text Needs Explicit accessibilityValue

`Text` with `.accessibilityIdentifier` is findable, but `.label` returns empty string. Set `.accessibilityValue` explicitly:

```swift
Text(displayString)
    .accessibilityIdentifier("display")
    .accessibilityValue(displayString)    // required for reading value in tests
```

Read it in tests via `.value`:
```swift
let text = app.staticTexts["display"].value as? String ?? ""
```

### Always Wait for Elements

The app window may not be rendered when XCUITest queries fire. Always wait:

```swift
override func setUpWithError() throws {
    app = XCUIApplication()
    app.launch()
    let element = app.staticTexts["display"]
    XCTAssertTrue(element.waitForExistence(timeout: 5))
}

// Before tapping:
let button = app.buttons["1"]
XCTAssertTrue(button.waitForExistence(timeout: 2))
button.tap()
```

### First-Run Automation Permissions

First-time XCUITest on macOS triggers system permission dialogs for accessibility/automation. Tests will fail until the user approves. This is not a code issue — inform the user if all UI tests fail on first run.

## Build and Test Workflow

1. `BuildProject` — verify compilation
2. `XcodeListNavigatorIssues` — check for warnings/errors
3. `RunAllTests` — run full suite
4. `RunSomeTests` with specific `targetName` + `testIdentifier` — run targeted tests
5. `GetBuildLog` with `severity: "error"` — diagnose build failures
6. `RenderPreview` — snapshot a SwiftUI `#Preview` for visual verification

Use `GetTestList` to discover available test identifiers before calling `RunSomeTests`.
