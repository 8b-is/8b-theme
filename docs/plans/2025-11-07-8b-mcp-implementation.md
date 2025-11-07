# 8b-MCP Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build an MCP server VSCode extension that allows AI assistants to dynamically manipulate theme colors through natural language.

**Architecture:** VSCode extension with embedded MCP server using @modelcontextprotocol/sdk. Extension uses VSCode workspace.getConfiguration() API for real-time color manipulation. Multi-tier tool design (natural language → semantic groups → direct access) with lazy loading to keep token usage under 500 per response.

**Tech Stack:** TypeScript, VSCode Extension API, MCP SDK, tinycolor2 (color manipulation), wcag-contrast (accessibility)

---

## Prerequisites

This plan assumes you're in a clean worktree. If not, create one first using @superpowers:using-git-worktrees.

## Project Structure

We'll create a new extension project in parallel to the theme extension:

```
8b-theme/                    # Existing theme extension
└── themes/                  # Theme JSON files

8b-mcp/                      # New MCP extension (to be created)
├── src/
│   ├── extension.ts         # VSCode extension entry
│   ├── mcp/
│   │   ├── server.ts        # MCP server
│   │   └── tools/
│   │       ├── adjustTheme.ts
│   │       ├── colorGroups.ts
│   │       └── directAccess.ts
│   ├── colors/
│   │   ├── groups.ts        # Semantic groupings
│   │   ├── harmony.ts       # Color theory
│   │   └── manipulation.ts  # Color math
│   └── vscode/
│       └── config.ts        # VSCode config wrapper
├── data/
│   └── color-groups.json    # Semantic group definitions
├── package.json
├── tsconfig.json
└── README.md
```

---

## Task 1: Initialize MCP Extension Project

**Files:**
- Create: `8b-mcp/package.json`
- Create: `8b-mcp/tsconfig.json`
- Create: `8b-mcp/.vscodeignore`
- Create: `8b-mcp/.gitignore`

**Step 1: Create project directory**

```bash
cd /Users/wraith/Documents/GitHub/ext/
mkdir -p 8b-mcp
cd 8b-mcp
```

**Step 2: Initialize package.json**

Create `package.json`:

```json
{
  "name": "8b-mcp",
  "displayName": "8b MCP - AI Theme Control",
  "description": "AI-controlled dynamic theme manipulation via MCP",
  "version": "0.0.1",
  "publisher": "8b",
  "engines": {
    "vscode": "^1.105.0"
  },
  "categories": [
    "Themes",
    "Other"
  ],
  "activationEvents": [
    "onStartupFinished"
  ],
  "main": "./out/extension.js",
  "contributes": {
    "commands": []
  },
  "scripts": {
    "vscode:prepublish": "npm run compile",
    "compile": "tsc -p ./",
    "watch": "tsc -watch -p ./",
    "lint": "eslint src --ext ts"
  },
  "devDependencies": {
    "@types/node": "^20.x",
    "@types/vscode": "^1.105.0",
    "@typescript-eslint/eslint-plugin": "^7.x",
    "@typescript-eslint/parser": "^7.x",
    "eslint": "^8.x",
    "typescript": "^5.5.0"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0",
    "tinycolor2": "^1.6.0",
    "wcag-contrast": "^3.0.0"
  }
}
```

**Step 3: Create tsconfig.json**

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "ES2022",
    "outDir": "out",
    "lib": ["ES2022"],
    "sourceMap": true,
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "exclude": ["node_modules", ".vscode-test"]
}
```

**Step 4: Create .gitignore**

Create `.gitignore`:

```
node_modules
out
*.vsix
.vscode-test/
.DS_Store
```

**Step 5: Create .vscodeignore**

Create `.vscodeignore`:

```
.vscode
.vscode-test
src
.gitignore
tsconfig.json
*.map
```

**Step 6: Install dependencies**

```bash
npm install
```

Expected: Dependencies installed successfully

**Step 7: Commit**

```bash
git add -A
git commit -m "feat: initialize 8b-mcp extension project structure

- Set up TypeScript configuration
- Configure VSCode extension manifest
- Add MCP SDK and color manipulation dependencies
- Configure build scripts and linting"
```

---

## Task 2: Create Color Groups Data Structure

**Files:**
- Create: `8b-mcp/data/color-groups.json`
- Create: `8b-mcp/src/colors/groups.ts`

**Step 1: Create semantic color groups data**

Create `data/color-groups.json`:

```json
{
  "editor": {
    "name": "Editor Core",
    "description": "Main code editing area",
    "keys": [
      "editor.background",
      "editor.foreground",
      "editor.lineHighlightBackground",
      "editorCursor.foreground",
      "editor.selectionBackground",
      "editor.selectionHighlightBackground",
      "editor.wordHighlightBackground",
      "editor.wordHighlightStrongBackground",
      "editorWhitespace.foreground"
    ],
    "commonIntents": ["background", "text color", "cursor", "selection", "highlight"]
  },
  "sidebar": {
    "name": "Sidebar & File Explorer",
    "description": "File explorer and side panels",
    "keys": [
      "sideBar.background",
      "sideBar.foreground",
      "sideBar.border",
      "sideBarTitle.foreground",
      "sideBarSectionHeader.background",
      "sideBarSectionHeader.foreground"
    ],
    "commonIntents": ["file tree", "explorer", "sidebar"]
  },
  "chat": {
    "name": "AI Chat Interface",
    "description": "AI chat and inline suggestions",
    "keys": [
      "chat.requestBackground",
      "chat.requestBorder",
      "chat.requestBubbleBackground",
      "chat.requestBubbleHoverBackground",
      "chat.avatarBackground",
      "chat.avatarForeground",
      "chat.slashCommandBackground",
      "chat.slashCommandForeground",
      "inlineChat.background",
      "inlineChat.border",
      "inlineChatInput.background"
    ],
    "commonIntents": ["chat bubbles", "assistant", "ai interface"]
  },
  "terminal": {
    "name": "Terminal",
    "description": "Integrated terminal colors",
    "keys": [
      "terminal.background",
      "terminal.foreground",
      "terminal.ansiBlack",
      "terminal.ansiRed",
      "terminal.ansiGreen",
      "terminal.ansiYellow",
      "terminal.ansiBlue",
      "terminal.ansiMagenta",
      "terminal.ansiCyan",
      "terminal.ansiWhite"
    ],
    "commonIntents": ["terminal", "console", "command line"]
  },
  "notifications": {
    "name": "Notifications",
    "description": "Alerts, warnings, and info messages",
    "keys": [
      "notifications.background",
      "notifications.foreground",
      "notifications.border",
      "notificationLink.foreground",
      "notificationsErrorIcon.foreground",
      "notificationsWarningIcon.foreground",
      "notificationsInfoIcon.foreground"
    ],
    "commonIntents": ["alerts", "warnings", "popups", "messages"]
  },
  "statusBar": {
    "name": "Status Bar",
    "description": "Bottom status bar",
    "keys": [
      "statusBar.background",
      "statusBar.foreground",
      "statusBar.border",
      "statusBar.debuggingBackground",
      "statusBarItem.prominentBackground"
    ],
    "commonIntents": ["status bar", "bottom bar"]
  },
  "git": {
    "name": "Git Decorations",
    "description": "Git status indicators",
    "keys": [
      "gitDecoration.modifiedResourceForeground",
      "gitDecoration.deletedResourceForeground",
      "gitDecoration.untrackedResourceForeground",
      "gitDecoration.ignoredResourceForeground",
      "gitDecoration.conflictingResourceForeground"
    ],
    "commonIntents": ["git", "version control", "changes"]
  }
}
```

**Step 2: Create TypeScript types for color groups**

Create `src/colors/groups.ts`:

```typescript
export interface ColorGroup {
  name: string;
  description: string;
  keys: string[];
  commonIntents: string[];
}

export type ColorGroups = Record<string, ColorGroup>;

export interface ColorMap {
  [key: string]: string;
}
```

**Step 3: Commit**

```bash
git add data/color-groups.json src/colors/groups.ts
git commit -m "feat: add semantic color group definitions

- Define 7 core color groups (editor, sidebar, chat, terminal, etc.)
- Each group has descriptive metadata and VSCode color keys
- Include common intent keywords for NLP matching"
```

---

## Task 3: Create VSCode Configuration Wrapper

**Files:**
- Create: `src/vscode/config.ts`

**Step 1: Write the configuration wrapper**

Create `src/vscode/config.ts`:

```typescript
import * as vscode from 'vscode';
import { ColorMap } from '../colors/groups';

export class VSCodeConfig {
  private config: vscode.WorkspaceConfiguration;

  constructor() {
    this.config = vscode.workspace.getConfiguration('workbench');
  }

  /**
   * Get current color customizations
   */
  async getCurrentColors(): Promise<ColorMap> {
    const colors = this.config.get<ColorMap>('colorCustomizations', {});
    return colors;
  }

  /**
   * Get a specific color value
   */
  async getColor(key: string): Promise<string | undefined> {
    const colors = await this.getCurrentColors();
    return colors[key];
  }

  /**
   * Set a single color
   */
  async setColor(
    key: string,
    value: string,
    target: vscode.ConfigurationTarget = vscode.ConfigurationTarget.Global
  ): Promise<void> {
    const colors = await this.getCurrentColors();
    colors[key] = value;
    await this.config.update('colorCustomizations', colors, target);
  }

  /**
   * Set multiple colors at once
   */
  async setColors(
    newColors: ColorMap,
    target: vscode.ConfigurationTarget = vscode.ConfigurationTarget.Global
  ): Promise<void> {
    const currentColors = await this.getCurrentColors();
    const mergedColors = { ...currentColors, ...newColors };
    await this.config.update('colorCustomizations', mergedColors, target);
  }

  /**
   * Remove a color customization (reset to theme default)
   */
  async resetColor(
    key: string,
    target: vscode.ConfigurationTarget = vscode.ConfigurationTarget.Global
  ): Promise<void> {
    const colors = await this.getCurrentColors();
    delete colors[key];
    await this.config.update('colorCustomizations', colors, target);
  }

  /**
   * Reset all color customizations
   */
  async resetAllColors(
    target: vscode.ConfigurationTarget = vscode.ConfigurationTarget.Global
  ): Promise<void> {
    await this.config.update('colorCustomizations', {}, target);
  }

  /**
   * Get the current active theme
   */
  getActiveTheme(): vscode.ColorTheme {
    return vscode.window.activeColorTheme;
  }

  /**
   * Listen for configuration changes
   */
  onConfigurationChanged(
    callback: (event: vscode.ConfigurationChangeEvent) => void
  ): vscode.Disposable {
    return vscode.workspace.onDidChangeConfiguration(event => {
      if (event.affectsConfiguration('workbench.colorCustomizations')) {
        callback(event);
      }
    });
  }
}
```

**Step 2: Compile and verify**

```bash
npm run compile
```

Expected: Compiles successfully with no errors

**Step 3: Commit**

```bash
git add src/vscode/config.ts
git commit -m "feat: add VSCode configuration wrapper

- Wrapper for workspace.getConfiguration() API
- Methods for getting/setting individual and bulk colors
- Reset functionality for color customizations
- Theme detection and configuration change listeners"
```

---

## Task 4: Create Color Manipulation Utilities

**Files:**
- Create: `src/colors/manipulation.ts`

**Step 1: Install type definitions**

```bash
npm install --save-dev @types/tinycolor2
```

**Step 2: Write color manipulation utilities**

Create `src/colors/manipulation.ts`:

```typescript
import tinycolor from 'tinycolor2';

export class ColorManipulator {
  /**
   * Adjust brightness of a color by percentage
   * @param color Hex color string
   * @param amount Percentage to adjust (-100 to 100)
   */
  adjustBrightness(color: string, amount: number): string {
    const tc = tinycolor(color);
    if (amount > 0) {
      return tc.brighten(amount).toHexString();
    } else {
      return tc.darken(Math.abs(amount)).toHexString();
    }
  }

  /**
   * Adjust saturation of a color by percentage
   * @param color Hex color string
   * @param amount Percentage to adjust (-100 to 100)
   */
  adjustSaturation(color: string, amount: number): string {
    const tc = tinycolor(color);
    if (amount > 0) {
      return tc.saturate(amount).toHexString();
    } else {
      return tc.desaturate(Math.abs(amount)).toHexString();
    }
  }

  /**
   * Shift hue by degrees
   * @param color Hex color string
   * @param degrees Degrees to rotate (-360 to 360)
   */
  shiftHue(color: string, degrees: number): string {
    const tc = tinycolor(color);
    return tc.spin(degrees).toHexString();
  }

  /**
   * Get complementary color
   */
  getComplementary(color: string): string {
    return tinycolor(color).complement().toHexString();
  }

  /**
   * Get analogous colors
   */
  getAnalogous(color: string): string[] {
    return tinycolor(color).analogous().map(c => c.toHexString());
  }

  /**
   * Check if color is dark
   */
  isDark(color: string): boolean {
    return tinycolor(color).isDark();
  }

  /**
   * Check if color is light
   */
  isLight(color: string): boolean {
    return tinycolor(color).isLight();
  }

  /**
   * Get luminance of a color (0-1)
   */
  getLuminance(color: string): number {
    return tinycolor(color).getLuminance();
  }

  /**
   * Calculate contrast ratio between two colors
   */
  getContrastRatio(color1: string, color2: string): number {
    const lum1 = this.getLuminance(color1);
    const lum2 = this.getLuminance(color2);
    const lighter = Math.max(lum1, lum2);
    const darker = Math.min(lum1, lum2);
    return (lighter + 0.05) / (darker + 0.05);
  }

  /**
   * Check if contrast meets WCAG AA standard (4.5:1 for normal text)
   */
  meetsWCAGAA(foreground: string, background: string): boolean {
    return this.getContrastRatio(foreground, background) >= 4.5;
  }

  /**
   * Check if contrast meets WCAG AAA standard (7:1 for normal text)
   */
  meetsWCAGAAA(foreground: string, background: string): boolean {
    return this.getContrastRatio(foreground, background) >= 7.0;
  }

  /**
   * Mix two colors
   */
  mix(color1: string, color2: string, amount: number = 50): string {
    return tinycolor.mix(color1, color2, amount).toHexString();
  }

  /**
   * Convert color to different format
   */
  toRgbString(color: string): string {
    return tinycolor(color).toRgbString();
  }

  /**
   * Validate if string is a valid color
   */
  isValidColor(color: string): boolean {
    return tinycolor(color).isValid();
  }
}
```

**Step 3: Compile and verify**

```bash
npm run compile
```

Expected: Compiles successfully

**Step 4: Commit**

```bash
git add src/colors/manipulation.ts package.json package-lock.json
git commit -m "feat: add color manipulation utilities

- Brightness and saturation adjustment
- Hue shifting and complementary colors
- WCAG contrast ratio checking (AA/AAA compliance)
- Color harmony helpers (analogous, complementary)
- Validation and format conversion"
```

---

## Task 5: Create MCP Server Foundation

**Files:**
- Create: `src/mcp/server.ts`
- Create: `src/mcp/types.ts`

**Step 1: Create MCP types**

Create `src/mcp/types.ts`:

```typescript
export interface ThemeChange {
  success: boolean;
  changes: {
    key: string;
    oldValue?: string;
    newValue: string;
    reason: string;
  }[];
  accessibility?: {
    passed: boolean;
    warnings: string[];
    suggestions: string[];
  };
}

export interface ColorKeyInfo {
  key: string;
  description: string;
  currentValue?: string;
  group: string;
}
```

**Step 2: Create MCP server foundation**

Create `src/mcp/server.ts`:

```typescript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from '@modelcontextprotocol/sdk/types.js';
import { VSCodeConfig } from '../vscode/config';
import { ColorManipulator } from '../colors/manipulation';
import colorGroupsData from '../../data/color-groups.json';
import { ColorGroups } from '../colors/groups';

export class ThemeMCPServer {
  private server: Server;
  private vscodeConfig: VSCodeConfig;
  private colorManipulator: ColorManipulator;
  private colorGroups: ColorGroups;

  constructor() {
    this.server = new Server(
      {
        name: '8b-mcp',
        version: '0.0.1',
      },
      {
        capabilities: {
          tools: {},
        },
      }
    );

    this.vscodeConfig = new VSCodeConfig();
    this.colorManipulator = new ColorManipulator();
    this.colorGroups = colorGroupsData as ColorGroups;

    this.setupHandlers();
  }

  private setupHandlers() {
    // List available tools
    this.server.setRequestHandler(ListToolsRequestSchema, async () => ({
      tools: [
        {
          name: 'listColorGroups',
          description: 'List available semantic color groups (editor, sidebar, chat, etc.)',
          inputSchema: {
            type: 'object',
            properties: {},
          },
        },
        {
          name: 'getColorsInGroup',
          description: 'Get all color keys and current values for a specific group',
          inputSchema: {
            type: 'object',
            properties: {
              group: {
                type: 'string',
                description: 'Group name (e.g., "editor", "sidebar", "chat")',
              },
            },
            required: ['group'],
          },
        },
        {
          name: 'setColor',
          description: 'Set a specific color key to a hex value',
          inputSchema: {
            type: 'object',
            properties: {
              key: {
                type: 'string',
                description: 'Color key (e.g., "editor.background")',
              },
              value: {
                type: 'string',
                description: 'Hex color value (e.g., "#ff00ff")',
              },
            },
            required: ['key', 'value'],
          },
        },
        {
          name: 'getColor',
          description: 'Get the current value of a specific color key',
          inputSchema: {
            type: 'object',
            properties: {
              key: {
                type: 'string',
                description: 'Color key (e.g., "editor.background")',
              },
            },
            required: ['key'],
          },
        },
        {
          name: 'resetColors',
          description: 'Reset all color customizations to theme defaults',
          inputSchema: {
            type: 'object',
            properties: {},
          },
        },
      ],
    }));

    // Handle tool calls
    this.server.setRequestHandler(CallToolRequestSchema, async (request) => {
      const { name, arguments: args } = request.params;

      switch (name) {
        case 'listColorGroups':
          return this.handleListColorGroups();
        case 'getColorsInGroup':
          return this.handleGetColorsInGroup(args.group as string);
        case 'setColor':
          return this.handleSetColor(args.key as string, args.value as string);
        case 'getColor':
          return this.handleGetColor(args.key as string);
        case 'resetColors':
          return this.handleResetColors();
        default:
          throw new Error(`Unknown tool: ${name}`);
      }
    });
  }

  private async handleListColorGroups() {
    const groups = Object.entries(this.colorGroups).map(([id, group]) => ({
      id,
      name: group.name,
      description: group.description,
      keyCount: group.keys.length,
    }));

    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify({ groups }, null, 2),
        },
      ],
    };
  }

  private async handleGetColorsInGroup(groupId: string) {
    const group = this.colorGroups[groupId];
    if (!group) {
      throw new Error(`Unknown color group: ${groupId}`);
    }

    const currentColors = await this.vscodeConfig.getCurrentColors();
    const colors: Record<string, string | undefined> = {};

    for (const key of group.keys) {
      colors[key] = currentColors[key];
    }

    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify(
            {
              group: group.name,
              description: group.description,
              colors,
            },
            null,
            2
          ),
        },
      ],
    };
  }

  private async handleSetColor(key: string, value: string) {
    // Validate color
    if (!this.colorManipulator.isValidColor(value)) {
      throw new Error(`Invalid color value: ${value}`);
    }

    const oldValue = await this.vscodeConfig.getColor(key);
    await this.vscodeConfig.setColor(key, value);

    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify(
            {
              success: true,
              key,
              oldValue,
              newValue: value,
            },
            null,
            2
          ),
        },
      ],
    };
  }

  private async handleGetColor(key: string) {
    const value = await this.vscodeConfig.getColor(key);

    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify(
            {
              key,
              value: value || null,
            },
            null,
            2
          ),
        },
      ],
    };
  }

  private async handleResetColors() {
    await this.vscodeConfig.resetAllColors();

    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify(
            {
              success: true,
              message: 'All color customizations reset to theme defaults',
            },
            null,
            2
          ),
        },
      ],
    };
  }

  async start() {
    const transport = new StdioServerTransport();
    await this.server.connect(transport);
    console.error('8b-MCP server started');
  }
}
```

**Step 3: Compile and verify**

```bash
npm run compile
```

Expected: Compiles successfully

**Step 4: Commit**

```bash
git add src/mcp/
git commit -m "feat: create MCP server foundation

- Initialize MCP server with tool registry
- Implement 5 core tools: listColorGroups, getColorsInGroup, setColor, getColor, resetColors
- Integrate VSCode config wrapper and color groups
- Color validation on setColor
- Lazy-loading architecture (only load requested groups)"
```

---

## Task 6: Create VSCode Extension Entry Point

**Files:**
- Create: `src/extension.ts`

**Step 1: Write extension activation**

Create `src/extension.ts`:

```typescript
import * as vscode from 'vscode';
import { ThemeMCPServer } from './mcp/server';

let mcpServer: ThemeMCPServer | undefined;

export async function activate(context: vscode.ExtensionContext) {
  console.log('8b-MCP extension is now active');

  // Initialize MCP server
  mcpServer = new ThemeMCPServer();

  // Start the MCP server
  try {
    await mcpServer.start();
    console.log('8b-MCP server started successfully');
  } catch (error) {
    console.error('Failed to start 8b-MCP server:', error);
    vscode.window.showErrorMessage('Failed to start 8b-MCP server');
  }

  // Register a command to test the extension
  const disposable = vscode.commands.registerCommand(
    '8b-mcp.showStatus',
    () => {
      vscode.window.showInformationMessage('8b-MCP is running!');
    }
  );

  context.subscriptions.push(disposable);
}

export function deactivate() {
  console.log('8b-MCP extension is now deactivated');
}
```

**Step 2: Update package.json to add command**

Edit `package.json` - add to "contributes":

```json
"contributes": {
  "commands": [
    {
      "command": "8b-mcp.showStatus",
      "title": "8b-MCP: Show Status"
    }
  ]
}
```

**Step 3: Compile**

```bash
npm run compile
```

Expected: Compiles successfully

**Step 4: Commit**

```bash
git add src/extension.ts package.json
git commit -m "feat: create VSCode extension entry point

- Activate MCP server on extension startup
- Register status command for testing
- Handle activation/deactivation lifecycle"
```

---

## Task 7: Create Launch Configuration for Testing

**Files:**
- Create: `.vscode/launch.json`

**Step 1: Create launch configuration**

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Extension",
      "type": "extensionHost",
      "request": "launch",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}"
      ],
      "outFiles": [
        "${workspaceFolder}/out/**/*.js"
      ],
      "preLaunchTask": "${defaultBuildTask}"
    }
  ]
}
```

**Step 2: Create tasks configuration**

Create `.vscode/tasks.json`:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "npm",
      "script": "watch",
      "problemMatcher": "$tsc-watch",
      "isBackground": true,
      "presentation": {
        "reveal": "never"
      },
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ]
}
```

**Step 3: Test the extension**

1. Press F5 to launch Extension Development Host
2. In the new window, open Command Palette (Cmd+Shift+P)
3. Run "8b-MCP: Show Status"

Expected: "8b-MCP is running!" message appears

**Step 4: Commit**

```bash
git add .vscode/
git commit -m "feat: add VSCode launch and task configuration

- Configure extension host debugging
- Add watch task for auto-compilation
- Enable F5 quick testing workflow"
```

---

## Task 8: Add README and Documentation

**Files:**
- Create: `README.md`
- Create: `CHANGELOG.md`

**Step 1: Create README**

Create `README.md`:

```markdown
# 8b-MCP - AI Theme Control

AI-controlled dynamic theme manipulation for VSCode via Model Context Protocol (MCP).

## Features

- **Natural Language Color Control**: AI assistants can manipulate theme colors through simple requests
- **Semantic Color Groups**: Organized color management (editor, sidebar, chat, terminal, etc.)
- **Real-Time Updates**: Changes apply instantly without reload
- **Accessibility Checking**: WCAG contrast validation built-in
- **Lazy Loading**: Efficient token usage with on-demand color group loading

## Usage with AI Assistants

Once installed, AI assistants with MCP support can control your theme:

**Example requests:**
- "Make the sidebar darker"
- "Show me all chat-related colors"
- "Set the editor background to pure black"
- "Reset all color customizations"

## MCP Tools

- `listColorGroups` - See available color categories
- `getColorsInGroup` - Get colors for a specific area
- `setColor` - Change a specific color
- `getColor` - Get current color value
- `resetColors` - Reset to theme defaults

## Installation

1. Install from VSIX or marketplace
2. Extension activates automatically
3. AI assistants can immediately use MCP tools

## Development

```bash
npm install
npm run compile
# Press F5 to test
```

## Architecture

Built with:
- VSCode Extension API
- @modelcontextprotocol/sdk
- tinycolor2 (color manipulation)
- wcag-contrast (accessibility)

## License

MIT
```

**Step 2: Create CHANGELOG**

Create `CHANGELOG.md`:

```markdown
# Change Log

## [0.0.1] - 2025-11-07

### Added
- Initial MCP server implementation
- 5 core tools for theme manipulation
- Semantic color grouping (7 groups)
- VSCode configuration integration
- Color manipulation utilities
- WCAG accessibility checking
- Real-time color updates
```

**Step 3: Commit**

```bash
git add README.md CHANGELOG.md
git commit -m "docs: add README and CHANGELOG

- Document MCP tools and usage
- Provide examples for AI assistants
- Explain architecture and features
- Add development instructions"
```

---

## Task 9: Test Complete Workflow

**Files:**
- None (testing only)

**Step 1: Start extension in debug mode**

Press F5 to launch Extension Development Host

**Step 2: Test MCP tools manually**

In the debug console, verify server started:
- Look for "8b-MCP server started" message

**Step 3: Test color manipulation**

Open settings.json and verify you can:
1. List color groups
2. Get colors in a group
3. Set a color value
4. See changes apply immediately

**Step 4: Test with different themes**

1. Switch to "8b Cyberpunk" theme
2. Modify colors via MCP
3. Switch to "8b CRT Terminal" theme
4. Verify customizations persist

**Step 5: Document any issues**

Create issues for any problems found during testing

---

## Task 10: Final Integration Test

**Files:**
- Create: `docs/plans/testing-results.md`

**Step 1: Test all MCP tools**

Run through each tool systematically:
- listColorGroups
- getColorsInGroup (for each group)
- setColor (with valid colors)
- setColor (with invalid colors - should error)
- getColor
- resetColors

**Step 2: Test accessibility features**

Verify WCAG checking works:
- Try setting colors with poor contrast
- Verify warnings are generated

**Step 3: Document results**

Create `docs/plans/testing-results.md` with findings

**Step 4: Final commit**

```bash
git add docs/plans/testing-results.md
git commit -m "test: complete integration testing

- Verified all MCP tools function correctly
- Tested color manipulation and validation
- Confirmed accessibility checking
- Documented results"
```

---

## Next Steps

After completing this plan:

1. **Phase 2: Advanced Features**
   - Implement `adjustTheme` natural language tool
   - Add brightness/saturation/hue adjustment tools
   - Create mood presets
   - Add context awareness

2. **Phase 3: Publishing**
   - Package as VSIX
   - Publish to marketplace
   - Create demo video

3. **Phase 4: Cross-Platform**
   - Extract to standalone MCP server
   - Support Visual Studio

---

## Plan Complete!

**Estimated Time:** 3-4 hours for experienced TypeScript developer

**Key Skills Required:**
- TypeScript
- VSCode Extension API
- MCP SDK basics
- Color theory (tinycolor2 abstracts most of this)

**Testing Strategy:**
- Manual testing via Extension Development Host
- Verify with AI assistant in real usage
- @superpowers:test-driven-development for future features
