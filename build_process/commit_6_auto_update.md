# Commit 6: Auto-Update Functionality

## Summary
Integrate electron-updater for automatic application updates with a comprehensive UI for checking, downloading, and installing updates.

## Changes

### Electron Main Process

#### `electron/main/updater.ts` (New)
Complete auto-update module:

**AppUpdater Class:**
- Extends EventEmitter for event-based notifications
- Configures electron-updater settings
- Manages update lifecycle

**Methods:**
- `setMainWindow(window)` - Set window for IPC
- `getStatus()` - Get current update status
- `checkForUpdates()` - Check for available updates
- `downloadUpdate()` - Download available update
- `quitAndInstall()` - Install update and restart
- `setChannel(channel)` - Set update channel (stable/beta/dev)
- `setAutoDownload(enable)` - Configure auto-download
- `getCurrentVersion()` - Get app version

**Update Status:**
- `idle` - No update activity
- `checking` - Checking for updates
- `available` - Update available
- `not-available` - Already on latest
- `downloading` - Download in progress
- `downloaded` - Ready to install
- `error` - Update failed

**IPC Handlers (registerUpdateHandlers):**
- `update:status` - Get current status
- `update:version` - Get app version
- `update:check` - Trigger update check
- `update:download` - Start download
- `update:install` - Install and restart
- `update:setChannel` - Change update channel
- `update:setAutoDownload` - Toggle auto-download

**Events forwarded to renderer:**
- `update:status-changed`
- `update:checking`
- `update:available`
- `update:not-available`
- `update:progress`
- `update:downloaded`
- `update:error`

#### `electron/main/index.ts`
- Import and register update handlers
- Auto-check for updates in production (10s delay)

#### `electron/preload/index.ts`
- Added all update IPC channels

### React Renderer

#### `src/stores/update.ts` (New)
Zustand store for update state:
- State: `status`, `currentVersion`, `updateInfo`, `progress`, `error`, `isInitialized`
- Actions: `init`, `checkForUpdates`, `downloadUpdate`, `installUpdate`, `setChannel`, `setAutoDownload`, `clearError`
- Listens for all update events from main process

#### `src/components/settings/UpdateSettings.tsx` (New)
Update UI component:
- Current version display
- Status indicator with icon
- Status text description
- Action buttons (Check/Download/Install/Retry)
- Download progress bar with transfer stats
- Update info card (version, date, release notes)
- Error details display

**Progress Display:**
- Transferred / Total bytes
- Transfer speed (bytes/second)
- Progress percentage bar

#### `src/components/ui/progress.tsx` (New)
Radix UI Progress component for download visualization.

#### `src/pages/Settings/index.tsx`
- Replaced manual update section with `UpdateSettings` component
- Added `useUpdateStore` for version display
- Removed unused state and effect hooks

## Technical Details

### Update Flow
```
App Start (Production)
       |
       v
  10s Delay
       |
       v
checkForUpdates()
       |
       +-- No Update --> status: 'not-available'
       |
       +-- Update Found --> status: 'available'
              |
              v
         [User Action]
              |
              v
       downloadUpdate()
              |
              v
       status: 'downloading'
       progress events
              |
              v
       status: 'downloaded'
              |
              v
         [User Action]
              |
              v
       quitAndInstall()
              |
              v
         App Restarts
```

### electron-updater Configuration
```typescript
autoUpdater.autoDownload = false;        // Manual download trigger
autoUpdater.autoInstallOnAppQuit = true; // Install on quit
autoUpdater.logger = customLogger;       // Console logging
```

### Update Channels
- `stable` - Production releases
- `beta` - Pre-release testing
- `dev` - Development builds

### Progress Info Interface
```typescript
interface ProgressInfo {
  total: number;           // Total bytes
  delta: number;           // Bytes since last event
  transferred: number;     // Bytes downloaded
  percent: number;         // 0-100
  bytesPerSecond: number;  // Transfer speed
}
```

### UI States
| Status | Icon | Action Button |
|--------|------|---------------|
| idle | RefreshCw (gray) | Check for Updates |
| checking | Loader2 (spin, blue) | Checking... (disabled) |
| available | Download (green) | Download Update |
| downloading | Download (pulse, blue) | Downloading... (disabled) |
| downloaded | CheckCircle2 (green) | Install & Restart |
| error | AlertCircle (red) | Retry |
| not-available | CheckCircle2 (green) | Check for Updates |

## Dependencies
- electron-updater ^6.3.9 (already installed)
- @radix-ui/react-progress ^1.1.1 (already installed)

## Version
v0.1.0-alpha (incremental)
