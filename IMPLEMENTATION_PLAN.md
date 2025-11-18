# Garmin Smart Analysis Plugin - Implementation Plan

## Overview

An Obsidian plugin that automatically analyzes your Garmin running activities using AI-powered insights. The plugin polls for new activities, performs comprehensive analysis using MCP servers, and updates your daily notes with insights and visualizations.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│ Obsidian Plugin (TypeScript)                                │
│                                                               │
│  ┌──────────────────┐                                        │
│  │ Activity Poller  │  Lightweight check:                    │
│  │ (Python Bridge)  │  "New runs today?"                     │
│  │                  │  → activity IDs only                   │
│  │ python-garmin    │                                        │
│  │ connect API      │                                        │
│  └────────┬─────────┘                                        │
│           │                                                   │
│           │ New activity detected                            │
│           ▼                                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Mastra Agent                                        │    │
│  │                                                      │    │
│  │  ┌────────────────────────────────────────────────┐ │    │
│  │  │ garmin-connect-mcp (MCP)                       │ │    │
│  │  │ - query_activities (get full details)          │ │    │
│  │  │ - get_activity_details                         │ │    │
│  │  │ - analyze_training_period                      │ │    │
│  │  │ - compare_activities                           │ │    │
│  │  │ - get_performance_metrics                      │ │    │
│  │  └────────────────────────────────────────────────┘ │    │
│  │                                                      │    │
│  │  ┌────────────────────────────────────────────────┐ │    │
│  │  │ mcp-server-chart (MCP)                         │ │    │
│  │  │ - create_line_chart (pace trends)              │ │    │
│  │  │ - create_bar_chart (comparisons)               │ │    │
│  │  │ - create_area_chart (heart rate zones)         │ │    │
│  │  └────────────────────────────────────────────────┘ │    │
│  └─────────────────────────────────────────────────────┘    │
│           │                                                   │
│           ▼                                                   │
│  ┌──────────────────┐                                        │
│  │ Note Writer      │  Update daily notes with               │
│  │                  │  insights + charts                     │
│  └──────────────────┘                                        │
└─────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

#### 1. Activity Poller (Python + TypeScript Bridge)
- **Purpose**: Lightweight polling to check for new activities
- **Technology**: `python-garminconnect` library
- **Output**: Activity IDs only (minimal data transfer)
- **Frequency**: Configurable (default: every 30 minutes)

#### 2. Mastra Agent
- **Purpose**: Orchestrate analysis and visualization
- **Technology**: Mastra AI framework with Claude
- **Capabilities**:
  - Autonomous decision making
  - Multi-tool coordination
  - Natural language insight generation

#### 3. MCP Servers

**garmin-connect-mcp:**
- Fetch full activity details
- Perform training analysis
- Compare activities
- Calculate performance metrics

**mcp-server-chart:**
- Generate 25+ chart types
- Create line charts (trends)
- Create bar charts (comparisons)
- Create area charts (distributions)

#### 4. Note Writer
- **Purpose**: Update Obsidian daily notes
- **Output**: Markdown with insights and embedded charts
- **Features**: Customizable templates, frontmatter metadata

## Implementation Phases

### Phase 1: Foundation & Setup

#### 1.1 Project Configuration

**Tasks:**
- Update `manifest.json`:
  ```json
  {
    "id": "garmin-smart-analysis",
    "name": "Garmin Smart Analysis",
    "version": "0.1.0",
    "minAppVersion": "0.15.0",
    "description": "AI-powered analysis of Garmin running activities",
    "author": "Your Name",
    "isDesktopOnly": true
  }
  ```

- Create directory structure:
  ```
  src/
    main.ts
    types.ts
    settings.ts
    commands/
      index.ts
    services/
      activity-poller.ts
      mcp-manager.ts
      analysis-agent.ts
      analysis-orchestrator.ts
      scheduler.ts
      note-writer.ts
    ui/
      settings-tab.ts
    utils/
  scripts/
    poll_activities.py
    requirements.txt
  ```

- Install dependencies:
  ```json
  {
    "dependencies": {
      "@mastra/core": "latest",
      "@mastra/mcp": "latest",
      "@ai-sdk/anthropic": "latest"
    }
  }
  ```

#### 1.2 Type Definitions (src/types.ts)

```typescript
export interface GarminSettings {
  garminEmail: string;
  garminPassword: string;
  anthropicApiKey: string;
  pythonPath: string;
  pollIntervalMinutes: number;
  lastCheckTime: string;
  noteTemplate: string;
}

export interface ActivityPollResult {
  hasNew: boolean;
  activityIds: string[];
  count: number;
}

export interface AnalysisResult {
  timestamp: string;
  insights: string;
  metricsTable: string;
  charts: ChartResult[];
}

export interface ChartResult {
  title: string;
  url: string;
  type: string;
}
```

### Phase 2: Lightweight Activity Polling

#### 2.1 Python Polling Script (scripts/poll_activities.py)

```python
#!/usr/bin/env python3
import sys
import json
from datetime import datetime
from garminconnect import Garmin

def poll_new_activities(email, password, last_check_timestamp):
    """
    Check for new activities since last_check_timestamp.
    Returns only activity IDs (minimal data).
    """
    try:
        client = Garmin(email, password)
        client.login()

        # Get activities from last check to now (lightweight query)
        activities = client.get_activities_by_date(
            last_check_timestamp,
            datetime.now().isoformat(),
            limit=10  # Only recent ones
        )

        # Return only IDs and basic info (minimal data)
        result = {
            'has_new': len(activities) > 0,
            'activity_ids': [str(a['activityId']) for a in activities],
            'count': len(activities)
        }

        print(json.dumps(result))
        sys.exit(0)

    except Exception as e:
        print(json.dumps({'error': str(e)}), file=sys.stderr)
        sys.exit(1)

if __name__ == '__main__':
    if len(sys.argv) != 4:
        print("Usage: poll_activities.py <email> <password> <last_check_timestamp>")
        sys.exit(1)

    poll_new_activities(sys.argv[1], sys.argv[2], sys.argv[3])
```

#### 2.2 Python Requirements (scripts/requirements.txt)

```
garminconnect>=0.2.0
```

#### 2.3 TypeScript Polling Bridge (src/services/activity-poller.ts)

```typescript
import { spawn } from 'child_process';
import { Notice } from 'obsidian';

export class ActivityPoller {
  constructor(
    private scriptsPath: string,
    private settings: GarminSettings
  ) {}

  async checkForNewActivities(
    lastCheckTime: string
  ): Promise<ActivityPollResult> {
    return new Promise((resolve, reject) => {
      const pythonPath = this.settings.pythonPath || 'python3';
      const scriptPath = `${this.scriptsPath}/poll_activities.py`;

      const process = spawn(pythonPath, [
        scriptPath,
        this.settings.garminEmail,
        this.settings.garminPassword,
        lastCheckTime
      ]);

      const out: string[] = [];
      const err: string[] = [];

      process.stdout.on('data', (data) => {
        out.push(data.toString());
      });

      process.stderr.on('data', (data) => {
        err.push(data.toString());
      });

      process.on('exit', (code) => {
        if (code !== 0) {
          const errorMsg = err.join('\n');
          new Notice(`Garmin polling failed: ${errorMsg}`);
          reject(new Error(errorMsg));
          return;
        }

        try {
          const result = JSON.parse(out.join(''));
          resolve(result);
        } catch (e) {
          reject(new Error(`Failed to parse polling result: ${e}`));
        }
      });
    });
  }
}
```

### Phase 3: MCP Integration for Analysis

#### 3.1 MCP Manager (src/services/mcp-manager.ts)

```typescript
import { MastraMCPClient } from "@mastra/mcp";

export class MCPManager {
  private garminMCP: MastraMCPClient;
  private chartMCP: MastraMCPClient;
  private garminTools: any;
  private chartTools: any;

  async connect() {
    // Initialize garmin-connect-mcp for data fetching AND analysis
    this.garminMCP = new MastraMCPClient({
      name: "garmin-analysis",
      server: {
        command: "uv",
        args: ["run", "garmin-connect-mcp"],
        env: process.env
      }
    });

    // Initialize mcp-server-chart for visualization
    this.chartMCP = new MastraMCPClient({
      name: "chart-generator",
      server: {
        command: "npx",
        args: ["-y", "@antv/mcp-server-chart"]
      }
    });

    await this.garminMCP.connect();
    await this.chartMCP.connect();

    this.garminTools = await this.garminMCP.tools();
    this.chartTools = await this.chartMCP.tools();
  }

  async disconnect() {
    await this.garminMCP?.disconnect();
    await this.chartMCP?.disconnect();
  }

  getToolsets() {
    return {
      garmin: this.garminTools,
      charts: this.chartTools
    };
  }
}
```

#### 3.2 Mastra Agent Configuration (src/services/analysis-agent.ts)

```typescript
import { Agent } from "@mastra/core/agent";
import { anthropic } from "@ai-sdk/anthropic";

export class AnalysisAgent {
  private agent: Agent;

  constructor(private apiKey: string) {
    this.agent = new Agent({
      name: "Run Analysis Agent",
      instructions: `You are an expert running performance analyst with access to Garmin activity data and visualization tools.

When given activity IDs, you should:
1. Fetch full activity details using garmin tools
2. Analyze performance metrics (pace, heart rate, elevation, training effect)
3. Compare with recent activities to identify trends and patterns
4. Create informative visualizations using chart tools
5. Generate actionable insights and training recommendations

Available tool categories:
- garmin:* - Query activities, get details, analyze training, compare performances
- charts:* - Create line charts, bar charts, area charts, and other visualizations

Focus on:
- Performance trends over time
- Training intensity and recovery patterns
- Heart rate zone distribution
- Pace consistency and improvements
- Comparative analysis with previous runs
- Actionable training recommendations`,
      model: anthropic("claude-3-5-sonnet-latest", {
        apiKey: this.apiKey
      })
    });
  }

  async analyzeActivities(activityIds: string[], toolsets: any) {
    const prompt = `New running activities detected: ${activityIds.join(', ')}

Please perform a comprehensive analysis:

1. Fetch full details for these activities
2. Compare with recent training history (last 30 days)
3. Analyze performance trends:
   - Pace progression
   - Heart rate patterns
   - Training load and recovery
   - Elevation profile (if applicable)

4. Create visualizations:
   - Pace trend chart (last 10 runs)
   - Heart rate zone distribution
   - Weekly distance/volume comparison
   - Any other relevant charts

5. Generate insights:
   - Key performance indicators
   - Training patterns and trends
   - Recommendations for improvement
   - Recovery and training load balance

Format the output as structured markdown suitable for an Obsidian daily note.`;

    return await this.agent.stream(prompt, { toolsets });
  }
}
```

### Phase 4: Analysis Orchestrator

#### 4.1 Analysis Orchestrator (src/services/analysis-orchestrator.ts)

```typescript
import { Notice } from 'obsidian';

export class AnalysisOrchestrator {
  constructor(
    private agent: AnalysisAgent,
    private mcpManager: MCPManager
  ) {}

  async processNewActivities(activityIds: string[]): Promise<AnalysisResult> {
    try {
      const toolsets = this.mcpManager.getToolsets();

      new Notice(`Analyzing ${activityIds.length} new activities...`);

      const response = await this.agent.analyzeActivities(
        activityIds,
        toolsets
      );

      // Collect streamed results
      const results = await this.parseAnalysisResults(response);

      new Notice('Analysis complete!');

      return results;
    } catch (error) {
      new Notice(`Analysis failed: ${error.message}`);
      throw error;
    }
  }

  private async parseAnalysisResults(stream: any): Promise<AnalysisResult> {
    let fullText = '';
    const charts: ChartResult[] = [];

    for await (const part of stream.fullStream) {
      if (part.type === 'text-delta') {
        fullText += part.textDelta;
      } else if (part.type === 'tool-result') {
        // Extract chart URLs from tool results
        if (part.toolName?.startsWith('charts:')) {
          charts.push({
            title: part.toolName,
            url: part.result?.url || '',
            type: part.toolName.replace('charts:', '')
          });
        }
      }
    }

    return {
      timestamp: new Date().toISOString(),
      insights: fullText,
      metricsTable: this.extractMetricsTable(fullText),
      charts
    };
  }

  private extractMetricsTable(text: string): string {
    // Extract markdown table from analysis text
    const tableRegex = /\|.*\|[\s\S]*?\n\|.*\|/g;
    const match = text.match(tableRegex);
    return match ? match[0] : '';
  }
}
```

### Phase 5: Background Processing Loop

#### 5.1 Scheduler Service (src/services/scheduler.ts)

```typescript
import { Notice } from 'obsidian';

export class Scheduler {
  private intervalId: number | null = null;
  private lastCheckTime: string;

  constructor(
    private poller: ActivityPoller,
    private orchestrator: AnalysisOrchestrator,
    private noteWriter: NoteWriter,
    private settings: GarminSettings
  ) {
    this.lastCheckTime = settings.lastCheckTime || new Date().toISOString();
  }

  async runPollingCycle() {
    try {
      // 1. Lightweight poll for new activities
      const pollResult = await this.poller.checkForNewActivities(
        this.lastCheckTime
      );

      // 2. If new activities detected, trigger analysis
      if (pollResult.hasNew) {
        new Notice(`Found ${pollResult.count} new activities!`);

        const analysis = await this.orchestrator.processNewActivities(
          pollResult.activityIds
        );

        // 3. Write to notes
        await this.noteWriter.updateDailyNote(analysis);
      }

      // 4. Update last check time
      this.lastCheckTime = new Date().toISOString();
      this.settings.lastCheckTime = this.lastCheckTime;

    } catch (error) {
      console.error('Polling cycle error:', error);
      new Notice(`Garmin polling error: ${error.message}`);
    }
  }

  start() {
    const intervalMs = this.settings.pollIntervalMinutes * 60 * 1000;

    // Run immediately on start
    this.runPollingCycle();

    // Then schedule regular polling
    this.intervalId = window.setInterval(
      () => this.runPollingCycle(),
      intervalMs
    );

    new Notice(`Garmin polling started (every ${this.settings.pollIntervalMinutes} min)`);
  }

  stop() {
    if (this.intervalId) {
      window.clearInterval(this.intervalId);
      this.intervalId = null;
    }
  }
}
```

### Phase 6: Note Writing

#### 6.1 Note Writer (src/services/note-writer.ts)

```typescript
import { App, TFile, moment } from 'obsidian';

export class NoteWriter {
  constructor(
    private app: App,
    private settings: GarminSettings
  ) {}

  async updateDailyNote(analysis: AnalysisResult) {
    const today = moment().format('YYYY-MM-DD');
    const dailyNotePath = `Daily Notes/${today}.md`;

    // Generate markdown content
    const content = this.formatAnalysisMarkdown(analysis);

    // Append to daily note
    await this.appendToNote(dailyNotePath, content);
  }

  private formatAnalysisMarkdown(analysis: AnalysisResult): string {
    const chartMarkdown = analysis.charts
      .map(c => `![${c.title}](${c.url})`)
      .join('\n\n');

    return `
## 🏃 Running Analysis

${analysis.insights}

### Performance Metrics
${analysis.metricsTable}

### Visualizations
${chartMarkdown}

---
*Analysis generated at ${moment(analysis.timestamp).format('YYYY-MM-DD HH:mm')}*
`;
  }

  private async appendToNote(path: string, content: string) {
    let file = this.app.vault.getAbstractFileByPath(path);

    if (!file) {
      // Create new daily note if it doesn't exist
      await this.app.vault.create(path, content);
    } else if (file instanceof TFile) {
      // Append to existing note
      const existing = await this.app.vault.read(file);
      await this.app.vault.modify(file, existing + '\n' + content);
    }
  }
}
```

### Phase 7: Settings & UI

#### 7.1 Settings Tab (src/ui/settings-tab.ts)

```typescript
import { App, PluginSettingTab, Setting } from 'obsidian';
import GarminAnalysisPlugin from '../main';

export class GarminSettingsTab extends PluginSettingTab {
  constructor(app: App, private plugin: GarminAnalysisPlugin) {
    super(app, plugin);
  }

  display(): void {
    const { containerEl } = this;
    containerEl.empty();

    containerEl.createEl('h2', { text: 'Garmin Smart Analysis Settings' });

    // Garmin credentials
    new Setting(containerEl)
      .setName('Garmin Email')
      .setDesc('Your Garmin Connect email')
      .addText(text => text
        .setPlaceholder('email@example.com')
        .setValue(this.plugin.settings.garminEmail)
        .onChange(async (value) => {
          this.plugin.settings.garminEmail = value;
          await this.plugin.saveSettings();
        }));

    new Setting(containerEl)
      .setName('Garmin Password')
      .setDesc('Your Garmin Connect password')
      .addText(text => {
        text.inputEl.type = 'password';
        text
          .setPlaceholder('••••••••')
          .setValue(this.plugin.settings.garminPassword)
          .onChange(async (value) => {
            this.plugin.settings.garminPassword = value;
            await this.plugin.saveSettings();
          });
      });

    // Anthropic API key
    new Setting(containerEl)
      .setName('Anthropic API Key')
      .setDesc('API key for Claude (get from console.anthropic.com)')
      .addText(text => {
        text.inputEl.type = 'password';
        text
          .setPlaceholder('sk-ant-...')
          .setValue(this.plugin.settings.anthropicApiKey)
          .onChange(async (value) => {
            this.plugin.settings.anthropicApiKey = value;
            await this.plugin.saveSettings();
          });
      });

    // Python path
    new Setting(containerEl)
      .setName('Python Path')
      .setDesc('Path to Python 3 executable')
      .addText(text => text
        .setPlaceholder('python3')
        .setValue(this.plugin.settings.pythonPath)
        .onChange(async (value) => {
          this.plugin.settings.pythonPath = value;
          await this.plugin.saveSettings();
        }));

    // Poll interval
    new Setting(containerEl)
      .setName('Poll Interval')
      .setDesc('How often to check for new activities (minutes)')
      .addText(text => text
        .setPlaceholder('30')
        .setValue(String(this.plugin.settings.pollIntervalMinutes))
        .onChange(async (value) => {
          const num = parseInt(value);
          if (!isNaN(num) && num > 0) {
            this.plugin.settings.pollIntervalMinutes = num;
            await this.plugin.saveSettings();
          }
        }));
  }
}
```

#### 7.2 Commands (src/commands/index.ts)

```typescript
import { Notice } from 'obsidian';
import GarminAnalysisPlugin from '../main';

export function registerCommands(plugin: GarminAnalysisPlugin) {
  // Force check for new runs
  plugin.addCommand({
    id: 'check-new-runs',
    name: 'Check for new runs now',
    callback: async () => {
      new Notice('Checking for new activities...');
      await plugin.scheduler.runPollingCycle();
    }
  });

  // View last analysis
  plugin.addCommand({
    id: 'view-last-analysis',
    name: 'View last analysis',
    callback: () => {
      new Notice(`Last check: ${plugin.settings.lastCheckTime}`);
    }
  });
}
```

### Phase 8: Plugin Lifecycle

#### 8.1 Main Plugin (src/main.ts)

```typescript
import { Plugin } from 'obsidian';
import { GarminSettings, DEFAULT_SETTINGS } from './types';
import { GarminSettingsTab } from './ui/settings-tab';
import { registerCommands } from './commands';
import { ActivityPoller } from './services/activity-poller';
import { MCPManager } from './services/mcp-manager';
import { AnalysisAgent } from './services/analysis-agent';
import { AnalysisOrchestrator } from './services/analysis-orchestrator';
import { NoteWriter } from './services/note-writer';
import { Scheduler } from './services/scheduler';

export default class GarminAnalysisPlugin extends Plugin {
  settings: GarminSettings;
  scheduler: Scheduler;
  private mcpManager: MCPManager;
  private poller: ActivityPoller;
  private agent: AnalysisAgent;
  private orchestrator: AnalysisOrchestrator;
  private noteWriter: NoteWriter;

  async onload() {
    await this.loadSettings();

    // Initialize services
    const scriptsPath = `${this.app.vault.configDir}/plugins/garmin-smart-analysis/scripts`;

    this.poller = new ActivityPoller(scriptsPath, this.settings);
    this.mcpManager = new MCPManager();
    this.agent = new AnalysisAgent(this.settings.anthropicApiKey);
    this.orchestrator = new AnalysisOrchestrator(this.agent, this.mcpManager);
    this.noteWriter = new NoteWriter(this.app, this.settings);
    this.scheduler = new Scheduler(
      this.poller,
      this.orchestrator,
      this.noteWriter,
      this.settings
    );

    // Connect to MCP servers
    await this.mcpManager.connect();

    // Start polling scheduler
    this.scheduler.start();

    // Add settings tab
    this.addSettingTab(new GarminSettingsTab(this.app, this));

    // Register commands
    registerCommands(this);

    // Add status bar
    const statusBarItem = this.addStatusBarItem();
    statusBarItem.setText('Garmin: Ready');
  }

  async onunload() {
    this.scheduler.stop();
    await this.mcpManager.disconnect();
  }

  async loadSettings() {
    this.settings = Object.assign({}, DEFAULT_SETTINGS, await this.loadData());
  }

  async saveSettings() {
    await this.saveData(this.settings);
  }
}
```

## Setup Instructions

### 1. Prerequisites

- Obsidian Desktop app
- Python 3.7+
- Node.js 16+
- Garmin Connect account
- Anthropic API key

### 2. Installation

```bash
# Clone the repository
cd /path/to/obsidian/vault/.obsidian/plugins
git clone https://github.com/yourusername/obsidian-garmin-smart-analysis

# Install dependencies
cd obsidian-garmin-smart-analysis
npm install

# Install Python dependencies
pip install -r scripts/requirements.txt

# Build the plugin
npm run build
```

### 3. Configuration

1. Enable the plugin in Obsidian Settings → Community Plugins
2. Open plugin settings
3. Enter your Garmin Connect credentials
4. Enter your Anthropic API key
5. Configure poll interval (default: 30 minutes)
6. Verify Python path (default: python3)

### 4. External MCP Servers

The plugin requires these MCP servers to be available:

**garmin-connect-mcp:**
```bash
# Install
cd /path/to/garmin-connect-mcp
uv sync

# Configure (create .env)
GARMIN_EMAIL=your-email@example.com
GARMIN_PASSWORD=your-password

# Test
uv run garmin-connect-mcp-auth
```

**mcp-server-chart:**
```bash
# Automatically installed via npx when plugin starts
# No manual setup required
```

## Usage

### Automatic Analysis

Once configured, the plugin will:
1. Poll for new activities every N minutes
2. Detect new runs automatically
3. Analyze performance using AI
4. Generate insights and visualizations
5. Update your daily notes

### Manual Commands

- **Cmd/Ctrl + P** → "Check for new runs now"
- **Cmd/Ctrl + P** → "View last analysis"

### Expected Output

Daily notes will be updated with:
- Performance insights
- Training recommendations
- Pace trend charts
- Heart rate zone analysis
- Weekly distance comparisons

## Troubleshooting

### Python Script Fails

```bash
# Test Python script manually
cd .obsidian/plugins/garmin-smart-analysis/scripts
python3 poll_activities.py "your-email" "your-password" "2024-01-01T00:00:00"
```

### MCP Connection Issues

Check that MCP servers are accessible:
```bash
# Test garmin-connect-mcp
uv run garmin-connect-mcp

# Test chart server
npx -y @antv/mcp-server-chart
```

### No Analysis Generated

1. Check Obsidian Developer Console (Cmd/Ctrl + Shift + I)
2. Verify Anthropic API key is valid
3. Ensure MCP servers are running
4. Check plugin settings for correct credentials

## Development

### Project Structure

```
src/
  main.ts              # Plugin entry point
  types.ts             # Type definitions
  settings.ts          # Settings management
  commands/            # Command implementations
  services/            # Core services (polling, analysis, etc.)
  ui/                  # UI components (settings tab)
  utils/               # Utility functions

scripts/
  poll_activities.py   # Python polling script
  requirements.txt     # Python dependencies
```

### Key Design Patterns

**Separation of Concerns:**
- Polling (Python) vs Analysis (MCP)
- Data fetching vs Insight generation
- Agent orchestration vs Note writing

**Efficient Resource Usage:**
- Lightweight polling (< 1 sec)
- On-demand MCP analysis
- Minimal data transfer

**Error Resilience:**
- Graceful degradation
- Retry logic
- User notifications

## Future Enhancements

- [ ] Support for other activity types (cycling, swimming)
- [ ] Custom analysis prompts
- [ ] Historical data analysis
- [ ] Training plan generation
- [ ] Mobile app support (if MCP becomes mobile-compatible)
- [ ] Integration with other fitness platforms

## License

MIT

## Contributing

Contributions welcome! Please read CONTRIBUTING.md for guidelines.
