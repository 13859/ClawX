# Commit 3: Setup Wizard

## Summary
Implement a functional multi-step setup wizard for first-run user onboarding, guiding users through environment checks, AI provider configuration, channel connection, and skill selection.

## Changes

### React Renderer

#### `src/pages/Setup/index.tsx`
Complete rewrite with functional implementation:
- **Step 0: Welcome** - Introduction to ClawX
- **Step 1: Runtime Check** - Verify Node.js, OpenClaw, and Gateway status
- **Step 2: Provider Configuration** - Select AI provider and enter API key
- **Step 3: Channel Connection** - Choose messaging channel (WhatsApp, Telegram, etc.)
- **Step 4: Skill Selection** - Toggle predefined skill bundles
- **Step 5: Complete** - Summary of configuration

New components:
- `WelcomeContent` - Welcome message and features overview
- `RuntimeContent` - Environment checks with real Gateway status
- `ProviderContent` - Provider selection with API key input and validation
- `ChannelContent` - Channel type selection with QR code placeholder
- `SkillsContent` - Skill bundle toggles
- `CompleteContent` - Configuration summary

Features:
- Animated transitions using Framer Motion
- Progress indicator with step navigation
- Dynamic `canProceed` state based on step requirements
- API key visibility toggle
- Simulated API key validation

#### `src/stores/settings.ts`
- Added `setupComplete: boolean` to state
- Added `markSetupComplete()` action
- Setup status persisted via Zustand persist middleware

#### `src/App.tsx`
- Added `useLocation` for route tracking
- Added redirect logic: if `setupComplete` is false, navigate to `/setup`
- Fixed `handleNavigate` callback signature for IPC events

## Technical Details

### Setup Flow
```
Welcome -> Runtime -> Provider -> Channel -> Skills -> Complete
   |          |          |          |          |          |
   v          v          v          v          v          v
 [Skip]   [Check]   [Validate]  [Select]   [Toggle]   [Finish]
   |          |          |          |          |          |
   +---------+----------+----------+----------+----------+
                                                         |
                                              markSetupComplete()
                                                         |
                                                 Navigate to /
```

### Provider Types
- Anthropic (Claude) - API key starts with `sk-ant-`
- OpenAI (GPT) - API key starts with `sk-`
- Google (Gemini) - Length validation
- Ollama (Local) - No API key required
- Custom - Configurable base URL

### Channel Types
- WhatsApp - QR code connection
- Telegram - Bot token
- Discord - Bot token
- Slack - OAuth/Bot token
- WeChat - QR code connection

### Skill Bundles
- Productivity - Calendar, reminders, notes
- Developer - Code assistance, git operations
- Information - Web search, news, weather
- Entertainment - Music, games, trivia

## UI/UX Features
- Gradient background with glassmorphism cards
- Step progress dots with completion indicators
- Animated page transitions
- Back/Skip/Next navigation
- Toast notifications on completion

## Version
v0.1.0-alpha (incremental)
