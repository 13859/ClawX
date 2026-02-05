# Commit 5: Channel Connection Flows

## Summary
Implement comprehensive channel management UI with multi-platform support, including QR code-based connection for WhatsApp/WeChat and token-based connection for Telegram/Discord/Slack.

## Changes

### React Renderer

#### `src/pages/Channels/index.tsx`
Complete rewrite with enhanced functionality:

**Main Components:**
- `Channels` - Main page with channel list, stats, and add dialog
- `ChannelCard` - Individual channel display with connect/disconnect/delete
- `AddChannelDialog` - Multi-step channel addition flow

**Features:**
- Channel statistics (Total, Connected, Disconnected)
- Gateway status warning when not running
- Supported channel type grid for quick add
- Connection-type specific flows:
  - QR Code: WhatsApp, WeChat
  - Token: Telegram, Discord, Slack

**AddChannelDialog Flow:**
1. Select channel type
2. View connection instructions
3. For QR: Generate and display QR code
4. For Token: Enter bot token
5. Optionally name the channel
6. Confirm and add

**Channel Info Configuration:**
```typescript
const channelInfo: Record<ChannelType, {
  description: string;
  connectionType: 'qr' | 'token' | 'oauth';
  instructions: string[];
  tokenLabel?: string;
  docsUrl?: string;
}>
```

#### `src/stores/channels.ts`
Enhanced channel store with new actions:
- `addChannel(params)` - Add new channel with type, name, token
- `deleteChannel(channelId)` - Remove channel
- `requestQrCode(channelType)` - Request QR code from Gateway
- `clearError()` - Clear error state

Improved error handling:
- Graceful fallback when Gateway unavailable
- Local channel creation for offline mode

#### `src/types/channel.ts`
No changes - existing types sufficient.

### Electron Main Process

#### `electron/utils/store.ts`
- Converted to dynamic imports for ESM compatibility
- All functions now async

#### `electron/main/window.ts`
- Converted to dynamic imports for ESM compatibility
- `getWindowState()` and `saveWindowState()` now async

## Technical Details

### Channel Connection Architecture
```
AddChannelDialog
       |
       +-- QR Flow
       |      |
       |      v
       |   requestQrCode() --> Gateway --> WhatsApp/WeChat API
       |      |                               |
       |      v                               v
       |   Display QR  <-- qrCode string <-- QR Generated
       |      |
       |      v
       |   User Scans --> Device Confirms --> channel:status event
       |
       +-- Token Flow
              |
              v
           Enter Token --> addChannel() --> Gateway --> Bot API
              |                               |
              v                               v
           Validate  <-- success/error <-- Connection Attempt
```

### Channel Types Configuration
| Type | Connection | Token Label | Docs |
|------|------------|-------------|------|
| WhatsApp | QR Code | - | WhatsApp FAQ |
| Telegram | Token | Bot Token | BotFather Docs |
| Discord | Token | Bot Token | Developer Portal |
| Slack | Token | Bot Token (xoxb-) | Slack API |
| WeChat | QR Code | - | - |

### Connection Instructions
Each channel type provides step-by-step instructions:
- WhatsApp: Open app > Settings > Linked Devices > Scan
- Telegram: @BotFather > /newbot > Copy token
- Discord: Developer Portal > Application > Bot > Token
- Slack: API Apps > Create App > OAuth > Install

### UI Components Used
- `Card`, `Button`, `Input`, `Label` - Base components
- `Separator` - Visual dividers
- `StatusBadge` - Connection status
- `LoadingSpinner` - Loading states
- Lucide icons: Plus, Radio, RefreshCw, Power, QrCode, etc.

### Error Handling
- Gateway offline: Create local channel, show warning
- Connection failed: Display error on ChannelCard
- Invalid token: Show validation error in dialog

## Version
v0.1.0-alpha (incremental)
