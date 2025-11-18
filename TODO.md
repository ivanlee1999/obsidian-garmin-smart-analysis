# Garmin Smart Analysis Plugin - Implementation TODO

## Project Status

- [x] Implementation plan created
- [ ] Development in progress
- [ ] Testing phase
- [ ] Production ready

Last Updated: 2025-01-17

---

## Phase 1: Foundation & Setup

### 1.1 Project Configuration
- [ ] Update `manifest.json`
  - [ ] Change id to "garmin-smart-analysis"
  - [ ] Update name and description
  - [ ] Set isDesktopOnly: true
  - [ ] Add author information
- [ ] Create directory structure
  - [ ] `src/` directory
  - [ ] `src/commands/` directory
  - [ ] `src/services/` directory
  - [ ] `src/ui/` directory
  - [ ] `src/utils/` directory
  - [ ] `scripts/` directory
- [ ] Install npm dependencies
  - [ ] Add `@mastra/core`
  - [ ] Add `@mastra/mcp`
  - [ ] Add `@ai-sdk/anthropic`
  - [ ] Run `npm install`
- [ ] Update `package.json`
  - [ ] Add build scripts
  - [ ] Configure TypeScript
- [ ] Update `.gitignore`
  - [ ] Ignore Python virtual environments
  - [ ] Ignore `.env` files
  - [ ] Ignore build artifacts

### 1.2 Type Definitions
- [ ] Create `src/types.ts`
  - [ ] Define `GarminSettings` interface
  - [ ] Define `ActivityPollResult` interface
  - [ ] Define `AnalysisResult` interface
  - [ ] Define `ChartResult` interface
  - [ ] Export `DEFAULT_SETTINGS` constant

### 1.3 Settings Management
- [ ] Create `src/settings.ts`
  - [ ] Implement settings load/save functions
  - [ ] Add validation for required fields
  - [ ] Add default values

---

## Phase 2: Lightweight Activity Polling

### 2.1 Python Polling Script
- [ ] Create `scripts/poll_activities.py`
  - [ ] Import required libraries (garminconnect, json, sys)
  - [ ] Implement `poll_new_activities()` function
  - [ ] Add Garmin Connect login logic
  - [ ] Add `get_activities_by_date()` call
  - [ ] Return only activity IDs (minimal data)
  - [ ] Add error handling with JSON error output
  - [ ] Add command-line argument parsing
  - [ ] Make script executable (`chmod +x`)
- [ ] Create `scripts/requirements.txt`
  - [ ] Add `garminconnect>=0.2.0`
- [ ] Test Python script manually
  - [ ] Install Python dependencies: `pip install -r scripts/requirements.txt`
  - [ ] Test with real credentials
  - [ ] Verify JSON output format
  - [ ] Test error scenarios

### 2.2 TypeScript Python Bridge
- [ ] Create `src/services/activity-poller.ts`
  - [ ] Import Node.js `child_process`
  - [ ] Implement `ActivityPoller` class
  - [ ] Implement `checkForNewActivities()` method
  - [ ] Add subprocess spawning logic
  - [ ] Add stdout/stderr data collection
  - [ ] Add JSON parsing from Python output
  - [ ] Add error handling and user notifications
  - [ ] Add timeout handling
- [ ] Test TypeScript bridge
  - [ ] Test successful poll
  - [ ] Test with no new activities
  - [ ] Test with new activities
  - [ ] Test error scenarios

---

## Phase 3: MCP Integration for Analysis

### 3.1 MCP Manager Service
- [ ] Create `src/services/mcp-manager.ts`
  - [ ] Import `MastraMCPClient` from `@mastra/mcp`
  - [ ] Implement `MCPManager` class
  - [ ] Add `garminMCP` client initialization
  - [ ] Add `chartMCP` client initialization
  - [ ] Implement `connect()` method
  - [ ] Implement `disconnect()` method
  - [ ] Implement `getToolsets()` method
  - [ ] Add connection error handling
  - [ ] Add reconnection logic
- [ ] Test MCP connections
  - [ ] Verify garmin-connect-mcp server is accessible
  - [ ] Verify mcp-server-chart can be spawned
  - [ ] Test tool retrieval
  - [ ] Test disconnect/cleanup

### 3.2 Mastra Agent Configuration
- [ ] Create `src/services/analysis-agent.ts`
  - [ ] Import `Agent` from `@mastra/core/agent`
  - [ ] Import `anthropic` from `@ai-sdk/anthropic`
  - [ ] Implement `AnalysisAgent` class
  - [ ] Configure agent with comprehensive instructions
  - [ ] Implement `analyzeActivities()` method
  - [ ] Add streaming response handling
  - [ ] Add error handling for API failures
- [ ] Test agent functionality
  - [ ] Test with sample activity IDs
  - [ ] Verify tool calling works
  - [ ] Test streaming responses
  - [ ] Validate output format

---

## Phase 4: Analysis Orchestrator

### 4.1 Analysis Orchestration Service
- [ ] Create `src/services/analysis-orchestrator.ts`
  - [ ] Implement `AnalysisOrchestrator` class
  - [ ] Implement `processNewActivities()` method
  - [ ] Add streaming result collection
  - [ ] Implement `parseAnalysisResults()` method
  - [ ] Extract charts from tool results
  - [ ] Implement `extractMetricsTable()` method
  - [ ] Add progress notifications
  - [ ] Add comprehensive error handling
- [ ] Test orchestration
  - [ ] Test with single activity
  - [ ] Test with multiple activities
  - [ ] Verify chart extraction
  - [ ] Verify metrics table extraction

---

## Phase 5: Background Processing Loop

### 5.1 Scheduler Service
- [ ] Create `src/services/scheduler.ts`
  - [ ] Implement `Scheduler` class
  - [ ] Implement `runPollingCycle()` method
  - [ ] Add poll → analyze → write workflow
  - [ ] Implement `start()` method with interval
  - [ ] Implement `stop()` method with cleanup
  - [ ] Add last check time tracking
  - [ ] Update settings after each cycle
  - [ ] Add error recovery logic
  - [ ] Add user notifications
- [ ] Test scheduler
  - [ ] Test immediate run on start
  - [ ] Test interval-based polling
  - [ ] Test stop/cleanup
  - [ ] Test error recovery

---

## Phase 6: Note Writing

### 6.1 Note Writer Service
- [ ] Create `src/services/note-writer.ts`
  - [ ] Import Obsidian App, TFile, moment
  - [ ] Implement `NoteWriter` class
  - [ ] Implement `updateDailyNote()` method
  - [ ] Implement `formatAnalysisMarkdown()` method
  - [ ] Implement `appendToNote()` method
  - [ ] Add daily note path resolution
  - [ ] Handle note creation if not exists
  - [ ] Handle appending to existing notes
  - [ ] Add frontmatter metadata support
- [ ] Test note writing
  - [ ] Test creating new daily note
  - [ ] Test appending to existing note
  - [ ] Verify markdown formatting
  - [ ] Test chart embedding

---

## Phase 7: Settings & UI

### 7.1 Settings Tab UI
- [ ] Create `src/ui/settings-tab.ts`
  - [ ] Import Obsidian PluginSettingTab, Setting
  - [ ] Implement `GarminSettingsTab` class
  - [ ] Add Garmin email input field
  - [ ] Add Garmin password input field (type: password)
  - [ ] Add Anthropic API key input field (type: password)
  - [ ] Add Python path input field
  - [ ] Add poll interval input field
  - [ ] Add input validation
  - [ ] Add save on change handlers
  - [ ] Add helpful descriptions for each setting
- [ ] Test settings UI
  - [ ] Verify all fields render correctly
  - [ ] Test saving settings
  - [ ] Test loading saved settings
  - [ ] Test validation

### 7.2 Commands
- [ ] Create `src/commands/index.ts`
  - [ ] Implement `registerCommands()` function
  - [ ] Add "Check for new runs now" command
  - [ ] Add "View last analysis" command
  - [ ] Add command callbacks
  - [ ] Add user notifications
- [ ] Test commands
  - [ ] Test command palette integration
  - [ ] Test manual polling trigger
  - [ ] Test last analysis display

### 7.3 Status Bar
- [ ] Add status bar item in main.ts
  - [ ] Show connection status
  - [ ] Show last poll time
  - [ ] Update on polling cycle
  - [ ] Add click handler for quick info

---

## Phase 8: Plugin Lifecycle

### 8.1 Main Plugin File
- [ ] Update `src/main.ts`
  - [ ] Import all services
  - [ ] Implement `onload()` method
    - [ ] Load settings
    - [ ] Initialize all services
    - [ ] Connect to MCP servers
    - [ ] Start scheduler
    - [ ] Add settings tab
    - [ ] Register commands
    - [ ] Add status bar item
  - [ ] Implement `onunload()` method
    - [ ] Stop scheduler
    - [ ] Disconnect MCP servers
    - [ ] Cleanup resources
  - [ ] Implement `loadSettings()` method
  - [ ] Implement `saveSettings()` method
  - [ ] Add error handling for initialization
- [ ] Test plugin lifecycle
  - [ ] Test enable plugin
  - [ ] Test disable plugin
  - [ ] Test reload plugin
  - [ ] Verify cleanup on unload

---

## Phase 9: Testing & Debugging

### 9.1 Unit Testing
- [ ] Set up testing framework (Jest/Vitest)
- [ ] Write tests for `activity-poller.ts`
  - [ ] Test successful polling
  - [ ] Test error handling
  - [ ] Test JSON parsing
- [ ] Write tests for `analysis-orchestrator.ts`
  - [ ] Test result parsing
  - [ ] Test chart extraction
  - [ ] Test metrics extraction
- [ ] Write tests for `note-writer.ts`
  - [ ] Test markdown formatting
  - [ ] Test note creation
  - [ ] Test note appending

### 9.2 Integration Testing
- [ ] Test end-to-end polling cycle
  - [ ] Setup test Garmin account
  - [ ] Create test activities
  - [ ] Verify polling detects activities
  - [ ] Verify analysis runs
  - [ ] Verify notes are updated
- [ ] Test MCP server interactions
  - [ ] Test garmin-connect-mcp tools
  - [ ] Test mcp-server-chart tools
  - [ ] Test tool calling from agent
- [ ] Test error scenarios
  - [ ] Test invalid credentials
  - [ ] Test MCP connection failures
  - [ ] Test API rate limiting
  - [ ] Test network timeouts

### 9.3 Manual Testing
- [ ] Test with real Garmin account
- [ ] Test with real running activities
- [ ] Verify analysis quality
- [ ] Verify chart generation
- [ ] Test on macOS
- [ ] Test on Windows
- [ ] Test on Linux

---

## Phase 10: Documentation & Polish

### 10.1 User Documentation
- [ ] Create comprehensive README.md
  - [ ] Add installation instructions
  - [ ] Add setup guide with screenshots
  - [ ] Add usage examples
  - [ ] Add troubleshooting section
  - [ ] Add FAQ
- [ ] Create example daily note template
- [ ] Create video tutorial (optional)

### 10.2 Developer Documentation
- [ ] Review and finalize IMPLEMENTATION_PLAN.md
- [ ] Add inline code comments
- [ ] Add JSDoc comments for public APIs
- [ ] Create CONTRIBUTING.md
- [ ] Document architecture decisions

### 10.3 Polish
- [ ] Add loading indicators
- [ ] Improve error messages
- [ ] Add user-friendly notifications
- [ ] Optimize performance
  - [ ] Add debouncing where needed
  - [ ] Optimize polling frequency
  - [ ] Cache analysis results
- [ ] Add configuration validation
- [ ] Add health checks for MCP servers

---

## Phase 11: External Dependencies Setup

### 11.1 garmin-connect-mcp Server
- [ ] Install garmin-connect-mcp
  - [ ] Clone repository or install via package manager
  - [ ] Run `uv sync`
  - [ ] Configure `.env` with Garmin credentials
  - [ ] Run authentication setup: `uv run garmin-connect-mcp-auth`
  - [ ] Test server: `uv run garmin-connect-mcp`
- [ ] Document setup in README

### 11.2 Python Environment
- [ ] Verify Python 3.7+ is installed
- [ ] Create virtual environment (optional)
  - [ ] `python3 -m venv venv`
  - [ ] Activate: `source venv/bin/activate`
- [ ] Install requirements: `pip install -r scripts/requirements.txt`
- [ ] Test Python script manually

### 11.3 Anthropic API
- [ ] Create Anthropic account
- [ ] Generate API key
- [ ] Test API key with simple request
- [ ] Document in README

---

## Phase 12: Deployment & Release

### 12.1 Build & Package
- [ ] Run production build: `npm run build`
- [ ] Test built plugin in Obsidian
- [ ] Verify all features work
- [ ] Create release package
- [ ] Test installation from release package

### 12.2 Version Control
- [ ] Review all changes
- [ ] Commit with descriptive messages
- [ ] Tag release version
- [ ] Push to GitHub

### 12.3 Release
- [ ] Create GitHub release
- [ ] Add release notes
- [ ] Upload plugin package
- [ ] Submit to Obsidian community plugins (optional)

---

## Future Enhancements (Post v1.0)

- [ ] Support for other activity types
  - [ ] Cycling analysis
  - [ ] Swimming analysis
  - [ ] General cardio activities
- [ ] Custom analysis prompts
  - [ ] User-defined analysis templates
  - [ ] Prompt library
- [ ] Historical data analysis
  - [ ] Analyze all past activities
  - [ ] Long-term trend analysis
  - [ ] Year-over-year comparisons
- [ ] Training plan generation
  - [ ] AI-powered training plans
  - [ ] Goal-based recommendations
- [ ] Advanced visualizations
  - [ ] Interactive charts
  - [ ] 3D elevation profiles
  - [ ] Heatmaps of training locations
- [ ] Integration with other platforms
  - [ ] Strava sync
  - [ ] Apple Health integration
  - [ ] Google Fit integration
- [ ] Mobile support
  - [ ] If MCP becomes mobile-compatible
  - [ ] Responsive UI for mobile devices

---

## Notes & Issues

### Known Issues
- [ ] List any known bugs or limitations
- [ ] Track workarounds

### Performance Considerations
- [ ] Monitor polling overhead
- [ ] Optimize MCP connection lifecycle
- [ ] Cache analysis results to reduce API calls

### Security Considerations
- [ ] Never commit `.env` files
- [ ] Store credentials securely
- [ ] Validate all user inputs
- [ ] Sanitize markdown output

---

## Checklist for Each Feature

When implementing each feature, ensure:
- [ ] Code follows TypeScript best practices
- [ ] Error handling is comprehensive
- [ ] User notifications are clear and helpful
- [ ] Code is documented with comments
- [ ] Unit tests are written (if applicable)
- [ ] Manual testing is performed
- [ ] README is updated if needed

---

## Daily Development Log

### 2025-01-17
- [x] Created implementation plan (IMPLEMENTATION_PLAN.md)
- [x] Created TODO list (TODO.md)
- [ ] Next: Begin Phase 1 implementation

### Future Dates
<!-- Add daily progress notes here -->

---

## Quick Reference

### Build Commands
```bash
npm run build        # Production build
npm run dev          # Development build with watch
npm run lint         # Run linter
```

### Test Commands
```bash
npm test             # Run all tests
npm run test:watch   # Run tests in watch mode
```

### Python Commands
```bash
pip install -r scripts/requirements.txt   # Install Python deps
python3 scripts/poll_activities.py [args] # Test polling script
```

### MCP Server Commands
```bash
uv run garmin-connect-mcp              # Start garmin-connect-mcp
npx -y @antv/mcp-server-chart          # Start chart server
```

---

**Total Tasks:** 150+
**Estimated Time:** 40-60 hours
**Complexity:** Medium-High
**Priority:** High
