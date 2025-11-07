# AI-Controlled Theme System Design

**Date:** 2025-11-07
**Project:** 8b-theme + 8b-mcp
**Status:** Design Complete - Ready for Implementation

## Vision

Create a system where AI assistants can dynamically manipulate VSCode theme colors in real-time through natural language, making the coding environment responsive to user needs and context.

## Project Components

### 1. 8b-theme (VSCode Extension)
Beautiful starter themes that serve as the foundation for AI customization.

**Themes:**
- **8b Cyberpunk** ✅ (Current - neon magenta/cyan retro future)
- **8b CRT Terminal** (Classic phosphor green on black)
- **8b Amber Terminal** (Warm amber/orange monochrome)
- **8b IBM Blue** (Blue phosphor nostalgia)

### 2. 8b-mcp (MCP Server Extension)
AI-controlled theme manipulation system integrated into VSCode.

## Architecture

### Phase 1: VSCode Extension with Embedded MCP (Recommended Start)

```
┌─────────────────────────────────────────────┐
│          VSCode Extension (8b-mcp)          │
│                                             │
│  ┌────────────────┐    ┌─────────────────┐ │
│  │  MCP Server    │◄───┤  VSCode API     │ │
│  │  (Embedded)    │    │  Integration    │ │
│  └────────────────┘    └─────────────────┘ │
│         ▲                      │            │
│         │                      ▼            │
│         │              ┌─────────────────┐  │
│         │              │ workspace.      │  │
│         │              │ getConfiguration│  │
│         │              │ .update()       │  │
│         │              └─────────────────┘  │
└─────────┼───────────────────────┼───────────┘
          │                       │
          │                       ▼
    ┌─────┴────────┐      ┌──────────────┐
    │  AI Assistant │      │ settings.json│
    │  (via MCP)    │      │ .vscode/     │
    └──────────────┘      └──────────────┘
```

**Key Technologies:**
- TypeScript (VSCode extension + MCP server)
- VSCode Extension API
- MCP SDK (@modelcontextprotocol/sdk)
- Color manipulation libraries (tinycolor2, chroma-js)

## MCP Tool Design

### Tier 1: Natural Language Interface (Primary)

```typescript
adjustTheme(description: string) → ThemeChange
```
**Purpose:** AI describes desired changes in plain English
**Examples:**
- "make the sidebar darker"
- "increase contrast for the terminal"
- "give the chat interface a purple tint"

**Returns:** What changed, preview of new colors, accessibility check

### Tier 2: Semantic Groups (Controlled Discovery)

```typescript
listColorGroups() → string[]
// Returns: ["editor", "sidebar", "terminal", "chat", "git", "debug", ...]

getColorsInGroup(group: string) → ColorMap
// Example: getColorsInGroup("chat") → 15 chat-related colors

setGroupColors(group: string, colors: Partial<ColorMap>)
// Bulk update a semantic area
```

**Purpose:** AI can explore specific areas without loading all 400+ color keys

**Semantic Groups:**
- `editor` - Main editing area (background, foreground, cursor, selection)
- `sidebar` - File explorer and panels
- `terminal` - Terminal colors and ANSI palette
- `chat` - AI chat interface (bubbles, avatars, code blocks)
- `notifications` - Alerts, warnings, info messages
- `statusBar` - Bottom status bar
- `git` - Git decorations and diff colors
- `debug` - Debug-specific UI elements
- `peek` - Peek views and widgets
- `search` - Find/replace highlighting
- ~30 total groups covering all 400+ VSCode color keys

### Tier 3: Direct Access (Power Users)

```typescript
searchColorKeys(query: string) → ColorKeyInfo[]
// Example: searchColorKeys("notification")
// Returns matching keys with descriptions

setColor(key: string, value: string)
getColor(key: string) → string

describeColorKey(key: string) → KeyDescription
// What does this key control? Current value? Related keys?

suggestKeysForIntent(intent: string) → ColorKeySuggestion[]
// AI: "I want to change error colors"
// Returns: relevant keys with descriptions
```

### Tier 4: Intelligent Adjustments

```typescript
adjustBrightness(amount: number, target?: string)
// Example: "Make everything 20% darker"

adjustSaturation(amount: number, target?: string)
// Pump up or tone down color intensity

shiftHue(degrees: number, target?: string)
// Rotate color wheel (greens → blues)

adjustContrast(mode: 'increase' | 'decrease' | 'auto')
// Smart contrast boosting with WCAG awareness

applyMood(mood: string)
// Moods: "focus", "relax", "energize", "night", "day"
// Uses color psychology research

applyPreset(name: string)
// Presets: "warm", "cool", "monochrome", "high-contrast", "colorblind-friendly"
```

### Tier 5: Context Awareness

```typescript
setColorForContext(context: string, colors: ColorMap)
// Different colors for debugging, testing, reviewing
// Example: Red theme when debugging

getActiveContext() → string
// What mode is VSCode in?

trackUserPreferences() → PreferenceProfile
// Learn which adjustments user keeps vs reverts
```

## Smart Features

### 1. Accessibility Guardian
- Automatic WCAG contrast ratio checking
- Warn when colors are too similar
- Suggest fixes for accessibility issues
- Support colorblind-friendly palettes

### 2. Color Harmony Engine
- Maintain color harmony when adjusting
- Use color theory (complementary, analogous, triadic)
- Prevent "mud" colors from bad combinations
- Suggest harmonious alternatives

### 3. Context Detection
- Detect debugging mode → apply alert colors
- Detect focus mode → reduce distractions
- Time-of-day auto-adjustments
- Workspace-specific preferences

### 4. Learning System
- Track which adjustments user keeps
- Learn preferences over time
- Suggest personalized presets
- Remember context-specific preferences

## Technical Implementation

### Extension Structure

```
8b-mcp/
├── src/
│   ├── extension.ts              # VSCode extension entry point
│   ├── mcp/
│   │   ├── server.ts             # MCP server implementation
│   │   ├── tools/
│   │   │   ├── adjustTheme.ts    # Natural language tool
│   │   │   ├── colorGroups.ts    # Semantic grouping
│   │   │   ├── directAccess.ts   # Direct color manipulation
│   │   │   ├── adjustments.ts    # Intelligent adjustments
│   │   │   └── context.ts        # Context awareness
│   │   └── types.ts              # TypeScript types
│   ├── colors/
│   │   ├── groups.ts             # Semantic group definitions
│   │   ├── harmony.ts            # Color harmony engine
│   │   ├── accessibility.ts      # WCAG checking
│   │   └── manipulation.ts       # Color math utilities
│   ├── state/
│   │   ├── preferences.ts        # User preference tracking
│   │   ├── history.ts            # Undo/redo support
│   │   └── contexts.ts           # Context-based presets
│   └── vscode/
│       ├── config.ts             # VSCode config API wrapper
│       └── theme.ts              # Theme detection & info
├── data/
│   ├── color-groups.json         # Semantic grouping data
│   ├── moods.json                # Mood-based presets
│   └── presets.json              # Named color presets
├── package.json
├── tsconfig.json
└── README.md
```

### VSCode API Integration

**Reading Configuration:**
```typescript
const config = vscode.workspace.getConfiguration('workbench');
const currentColors = config.get<Record<string, string>>('colorCustomizations', {});
const currentTheme = vscode.window.activeColorTheme;
```

**Updating Colors:**
```typescript
await config.update(
  'workbench.colorCustomizations',
  {
    ...currentColors,
    'sideBar.background': '#2d1b3d',
    'sideBar.foreground': '#b0bec5'
  },
  vscode.ConfigurationTarget.Global // or Workspace, WorkspaceFolder
);
```

**Listening for Changes:**
```typescript
vscode.workspace.onDidChangeConfiguration(event => {
  if (event.affectsConfiguration('workbench.colorCustomizations')) {
    // React to external color changes
    updateMCPState();
  }
});

vscode.window.onDidChangeActiveColorTheme(theme => {
  // User switched base theme
  recalculateCustomizations();
});
```

## Data Structures

### Color Group Definition
```typescript
interface ColorGroup {
  id: string;
  name: string;
  description: string;
  keys: string[];                    // VSCode color keys
  commonIntents: string[];           // Natural language mappings
  relatedGroups: string[];           // Semantic relationships
}
```

### Theme Change Result
```typescript
interface ThemeChange {
  success: boolean;
  changes: {
    key: string;
    oldValue: string;
    newValue: string;
    reason: string;
  }[];
  accessibility: {
    passed: boolean;
    warnings: string[];
    suggestions: string[];
  };
  preview?: string;                  // Visual preview if possible
}
```

### User Preferences
```typescript
interface PreferenceProfile {
  favoriteColors: Record<string, string>;
  frequentAdjustments: AdjustmentHistory[];
  contextPresets: Record<string, ColorMap>;
  avoidedColors: string[];           // Colors user always reverts
  contrastPreference: 'low' | 'medium' | 'high';
  moodPreferences: Record<string, number>;  // Usage frequency
}
```

## CRT Theme Specification

### 8b CRT Terminal Theme

**Inspiration:** Classic CRT terminals with P3 phosphor green

**Core Palette:**
- Background: `#000000` (pure black, powered-off CRT)
- Foreground: `#33ff33` (P3 phosphor green)
- Cursor: `#ffffff` (bright electron beam)
- Selection: `#33ff3333` (subtle green glow)

**Intensity Levels:**
- Dim: `#1a8a1a` (inactive elements, comments)
- Medium: `#28cc28` (strings, standard text)
- Bright: `#3fff3f` (keywords, functions)
- Very Bright: `#80ff80` (active selection, focus)

**UI Colors:**
- Active borders: `#33ff33`
- Inactive borders: `#0a3a0a`
- Line highlight: `#00220011` (scanline effect)
- Glow effects: `#33ff3344` (subtle shadows)

**Special Features:**
- Terminal: Full phosphor immersion
- Chat interface: Green-tinted bubbles
- Notifications: Green-bordered alerts
- Status bar: Classic terminal aesthetic

**Error Handling:**
- Errors: `#ff3333` (slight red tint on phosphor)
- Warnings: `#ffff33` (yellow phosphor)
- Info: Standard green

## Future Enhancements

### Phase 2: Standalone MCP Server
Extract MCP functionality into standalone server for:
- Cross-editor support (Visual Studio, etc.)
- Remote theme control
- Multi-instance synchronization

### Phase 3: Visual Preview System
- Real-time color preview in chat
- Before/after comparisons
- Screenshot-based suggestions

### Phase 4: AI Theme Generator
- Generate complete themes from descriptions
- "Make me a theme based on ocean sunset"
- Export generated themes as files

### Phase 5: Community Features
- Share color adjustments
- Import community presets
- Theme marketplace integration

## Success Metrics

- **Usability:** AI can make 90%+ of color adjustments without human clarification
- **Performance:** Color updates apply in <100ms
- **Accessibility:** All AI suggestions maintain WCAG AA contrast ratios
- **Learning:** System improves suggestions after 10+ interactions
- **Token Efficiency:** Average MCP response <500 tokens

## Testing Strategy

1. **Unit Tests:** Color manipulation, harmony checks, accessibility validation
2. **Integration Tests:** VSCode API interactions, MCP protocol compliance
3. **User Testing:** Natural language understanding accuracy
4. **Accessibility Testing:** WCAG compliance verification
5. **Performance Testing:** Update latency, memory usage

## Dependencies

**Required:**
- `@types/vscode`: ^1.105.0
- `@modelcontextprotocol/sdk`: latest
- `tinycolor2`: ^1.6.0 (color manipulation)
- `wcag-contrast`: ^3.0.0 (accessibility checking)

**Development:**
- `@types/node`: latest
- `typescript`: ^5.0.0
- `esbuild`: latest (bundling)
- `eslint`: latest

## Rollout Plan

1. **Week 1:** Create CRT theme variants, set up extension project
2. **Week 2:** Implement core MCP tools (Tier 1-2)
3. **Week 3:** Add smart features (accessibility, harmony)
4. **Week 4:** Testing, documentation, polish
5. **Week 5:** Alpha release, gather feedback
6. **Week 6+:** Iterate based on real-world usage

## Notes

- Keep MCP responses under 500 tokens (lazy-loading strategy)
- All color changes must pass WCAG AA minimum
- Support undo/redo for all operations
- Maintain compatibility with VSCode theme system
- Document all semantic groups for AI understanding
- Remember: Hue trusts us to make this awesome! 🎨✨
