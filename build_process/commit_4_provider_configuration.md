# Commit 4: Provider Configuration

## Summary
Implement secure API key storage using Electron's safeStorage API and create a comprehensive provider management UI for configuring AI model providers.

## Changes

### Electron Main Process

#### `electron/utils/secure-storage.ts` (New)
Secure storage utility using Electron's safeStorage:
- `isEncryptionAvailable()` - Check if system keychain is available
- `storeApiKey(providerId, apiKey)` - Encrypt and store API key
- `getApiKey(providerId)` - Decrypt and retrieve API key
- `deleteApiKey(providerId)` - Remove stored API key
- `hasApiKey(providerId)` - Check if key exists
- `listStoredKeyIds()` - List all stored provider IDs

Provider configuration management:
- `saveProvider(config)` - Save provider configuration
- `getProvider(providerId)` - Get single provider
- `getAllProviders()` - Get all providers
- `deleteProvider(providerId)` - Delete provider and key
- `setDefaultProvider(providerId)` - Set default provider
- `getDefaultProvider()` - Get default provider ID
- `getProviderWithKeyInfo(providerId)` - Get provider with masked key
- `getAllProvidersWithKeyInfo()` - Get all providers with key info

Note: Uses dynamic `import('electron-store')` for ESM compatibility.

#### `electron/main/ipc-handlers.ts`
New `registerProviderHandlers()` function with IPC handlers:
- `provider:encryptionAvailable`
- `provider:list` / `provider:get` / `provider:save` / `provider:delete`
- `provider:setApiKey` / `provider:deleteApiKey` / `provider:hasApiKey` / `provider:getApiKey`
- `provider:setDefault` / `provider:getDefault`
- `provider:validateKey` - Basic format validation

#### `electron/preload/index.ts`
Added all provider IPC channels to valid channels list.

### React Renderer

#### `src/stores/providers.ts` (New)
Zustand store for provider management:
- State: `providers`, `defaultProviderId`, `loading`, `error`
- Actions: `fetchProviders`, `addProvider`, `updateProvider`, `deleteProvider`
- Actions: `setApiKey`, `deleteApiKey`, `setDefaultProvider`, `validateApiKey`, `getApiKey`

#### `src/components/settings/ProvidersSettings.tsx` (New)
Provider management UI component:
- `ProvidersSettings` - Main component orchestrating provider list
- `ProviderCard` - Individual provider display with actions
- `AddProviderDialog` - Modal for adding new providers

Features:
- Provider type icons (Anthropic, OpenAI, Google, Ollama, Custom)
- Masked API key display (first 4 + last 4 characters)
- Set default provider
- Enable/disable providers
- Edit/delete functionality
- API key validation on add

#### `src/pages/Settings/index.tsx`
- Added AI Providers section with `ProvidersSettings` component
- Added `Key` icon import

## Technical Details

### Security Architecture
```
Renderer Process                    Main Process
      |                                  |
      | provider:setApiKey               |
      |--------------------------------->|
      |                                  | safeStorage.encryptString()
      |                                  |         |
      |                                  |         v
      |                                  |  System Keychain
      |                                  |         |
      |                                  |  electron-store
      |                                  |  (encrypted base64)
      |                                  |
      | provider:getApiKey               |
      |--------------------------------->|
      |                                  | safeStorage.decryptString()
      |<---------------------------------|
      | (plaintext)                      |
```

### Provider Configuration Schema
```typescript
interface ProviderConfig {
  id: string;
  name: string;
  type: 'anthropic' | 'openai' | 'google' | 'ollama' | 'custom';
  baseUrl?: string;
  model?: string;
  enabled: boolean;
  createdAt: string;
  updatedAt: string;
}
```

### Key Validation Rules
- Anthropic: Must start with `sk-ant-`
- OpenAI: Must start with `sk-`
- Google: Minimum 20 characters
- Ollama: No key required
- Custom: No validation

### ESM Compatibility
electron-store v10+ is ESM-only. Solution: dynamic imports in main process.

```typescript
let store: any = null;
async function getStore() {
  if (!store) {
    const Store = (await import('electron-store')).default;
    store = new Store({ /* config */ });
  }
  return store;
}
```

## Version
v0.1.0-alpha (incremental)
