---
name: electron-sveltekit
description: >
  Electron patterns for building cross-platform desktop applications with a modern SvelteKit UI layer.
  Trigger: When building desktop apps with Electron and using SvelteKit for the renderer UI.
metadata:
  author: gentleman-programming
  version: "2.0"
  source: "https://github.com/Gentleman-Programming/Gentleman-Skills/tree/main/community/electron"
  ui_layer: "sveltekit"
---

## When to Use

Load this skill when:
- Building cross-platform desktop applications
- Working with Electron's main and renderer processes
- Implementing IPC (Inter-Process Communication)
- Integrating native OS features (menus, notifications, file system)
- Building the renderer with SvelteKit (Svelte 5 runes)
- Configuring auto-updates and app distribution

## Critical Patterns

### Pattern 1: Project Structure (Electron + SvelteKit)

```text
src/
|-- main/                    # Main process (Node.js)
|   |-- index.ts             # Entry point
|   |-- ipc/                 # IPC handlers
|   |   |-- handlers.ts
|   |   `-- channels.ts      # Type-safe channel names
|   |-- services/            # Native services
|   |   |-- store.ts         # electron-store
|   |   `-- updater.ts       # auto-updater
|   `-- windows/             # Window management
|       `-- main-window.ts
|-- preload/                 # Preload scripts
|   `-- index.ts             # Expose safe APIs via contextBridge
|-- shared/                  # Shared IPC contracts
|   `-- ipc.ts
`-- renderer/                # SvelteKit renderer app
    |-- src/
    |   |-- app.d.ts
    |   |-- app.html
    |   |-- lib/
    |   |   `-- electron/
    |   |       `-- ipc.svelte.ts
    |   `-- routes/
    |       |-- +layout.ts
    |       `-- +page.svelte
    |-- static/
    |-- svelte.config.js
    |-- tsconfig.json
    `-- vite.config.ts
```

### Pattern 2: Secure IPC Communication

Always use `contextBridge` for secure communication:

```typescript
// preload/index.ts
import { contextBridge, ipcRenderer } from 'electron';
import type { ElectronAPI, IpcChannels } from '../shared/ipc';

const electronAPI: ElectronAPI = {
  // One-way: renderer -> main
  send: <T extends keyof IpcChannels>(channel: T, data: IpcChannels[T]['request']) => {
    ipcRenderer.send(channel, data);
  },

  // Two-way: renderer -> main -> renderer
  invoke: <T extends keyof IpcChannels>(
    channel: T,
    data: IpcChannels[T]['request']
  ): Promise<IpcChannels[T]['response']> => {
    return ipcRenderer.invoke(channel, data);
  },

  // Listen: main -> renderer
  on: <T extends keyof IpcChannels>(
    channel: T,
    callback: (data: IpcChannels[T]['response']) => void
  ) => {
    const subscription = (_: Electron.IpcRendererEvent, data: IpcChannels[T]['response']) => {
      callback(data);
    };
    ipcRenderer.on(channel, subscription);
    return () => ipcRenderer.removeListener(channel, subscription);
  }
};

contextBridge.exposeInMainWorld('electron', electronAPI);
```

### Pattern 3: Type-Safe IPC Contracts

Define all channels with request/response types:

```typescript
// shared/ipc.ts
export interface IpcChannels {
  'app:get-version': {
    request: void;
    response: string;
  };
  'file:read': {
    request: { path: string };
    response: { content: string } | { error: string };
  };
  'file:write': {
    request: { path: string; content: string };
    response: { success: boolean };
  };
  'dialog:open-file': {
    request: { filters?: Electron.FileFilter[] };
    response: string | null;
  };
  'store:get': {
    request: { key: string };
    response: unknown;
  };
  'store:set': {
    request: { key: string; value: unknown };
    response: void;
  };
}

export type ElectronAPI = {
  send<T extends keyof IpcChannels>(channel: T, data: IpcChannels[T]['request']): void;
  invoke<T extends keyof IpcChannels>(
    channel: T,
    data: IpcChannels[T]['request']
  ): Promise<IpcChannels[T]['response']>;
  on<T extends keyof IpcChannels>(
    channel: T,
    callback: (data: IpcChannels[T]['response']) => void
  ): () => void;
};

declare global {
  interface Window {
    electron: ElectronAPI;
  }
}
```

### Pattern 4: SvelteKit Renderer Setup for Electron

Treat the renderer as a SvelteKit SPA in production so Electron can load from `file://`:

```typescript
// renderer/src/routes/+layout.ts
export const ssr = false;
export const prerender = true;
```

```javascript
// renderer/svelte.config.js
import adapter from '@sveltejs/adapter-static';

const config = {
  kit: {
    adapter: adapter({
      pages: 'build',
      assets: 'build',
      fallback: 'index.html'
    }),
    prerender: {
      entries: ['*']
    }
  }
};

export default config;
```

## Code Examples

### Example 1: Main Process Setup

```typescript
// main/index.ts
import { app, BrowserWindow } from 'electron';
import path from 'path';
import { registerIpcHandlers } from './ipc/handlers';

let mainWindow: BrowserWindow | null = null;

async function createWindow() {
  mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    minWidth: 800,
    minHeight: 600,
    webPreferences: {
      preload: path.join(__dirname, '../preload/index.js'),
      contextIsolation: true,
      nodeIntegration: false,
      sandbox: true
    },
    titleBarStyle: process.platform === 'darwin' ? 'hiddenInset' : 'default',
    trafficLightPosition: { x: 15, y: 10 }
  });

  registerIpcHandlers();

  if (!app.isPackaged) {
    await mainWindow.loadURL('http://localhost:5173');
    mainWindow.webContents.openDevTools({ mode: 'detach' });
  } else {
    await mainWindow.loadFile(path.join(__dirname, '../renderer/build/index.html'));
  }

  mainWindow.on('closed', () => {
    mainWindow = null;
  });
}

app.whenReady().then(createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') app.quit();
});

app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) createWindow();
});
```

### Example 2: IPC Handlers

```typescript
// main/ipc/handlers.ts
import { app, dialog, ipcMain } from 'electron';
import fs from 'node:fs/promises';
import Store from 'electron-store';

const store = new Store();

export function registerIpcHandlers() {
  ipcMain.handle('app:get-version', () => app.getVersion());

  ipcMain.handle('file:read', async (_, { path }) => {
    try {
      const content = await fs.readFile(path, 'utf-8');
      return { content };
    } catch (error) {
      return { error: (error as Error).message };
    }
  });

  ipcMain.handle('file:write', async (_, { path, content }) => {
    try {
      await fs.writeFile(path, content, 'utf-8');
      return { success: true };
    } catch {
      return { success: false };
    }
  });

  ipcMain.handle('dialog:open-file', async (_, { filters }) => {
    const result = await dialog.showOpenDialog({
      properties: ['openFile'],
      filters: filters || [{ name: 'All Files', extensions: ['*'] }]
    });

    return result.canceled ? null : result.filePaths[0];
  });

  ipcMain.handle('store:get', (_, { key }) => store.get(key));
  ipcMain.handle('store:set', (_, { key, value }) => {
    store.set(key, value);
  });
}
```

### Example 3: SvelteKit IPC Layer (Runes)

```typescript
// renderer/src/lib/electron/ipc.svelte.ts
import type { IpcChannels } from '../../../../shared/ipc';

type Channel = keyof IpcChannels;

export function createIpcQuery<T extends Channel>(
  channel: T,
  request: IpcChannels[T]['request']
) {
  const state = $state({
    data: null as IpcChannels[T]['response'] | null,
    loading: false,
    error: null as string | null
  });

  const run = async () => {
    state.loading = true;
    state.error = null;

    try {
      state.data = await window.electron.invoke(channel, request);
    } catch (error) {
      state.error = error instanceof Error ? error.message : 'Unknown IPC error';
    } finally {
      state.loading = false;
    }
  };

  return { state, run };
}

export function listenIpc<T extends Channel>(
  channel: T,
  callback: (data: IpcChannels[T]['response']) => void
) {
  return window.electron.on(channel, callback);
}
```

```svelte
<!-- renderer/src/routes/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { createIpcQuery } from '$lib/electron/ipc.svelte';

  const versionQuery = createIpcQuery('app:get-version', undefined);

  onMount(() => {
    versionQuery.run();
  });
</script>

<main>
  <h1>Electron + SvelteKit</h1>

  {#if versionQuery.state.loading}
    <p>Loading app version...</p>
  {:else if versionQuery.state.error}
    <p class="error">{versionQuery.state.error}</p>
  {:else if versionQuery.state.data}
    <p>Version: {versionQuery.state.data}</p>
  {/if}
</main>
```

### Example 4: Auto-Updater Setup

```typescript
// main/services/updater.ts
import { autoUpdater } from 'electron-updater';
import log from 'electron-log';
import type { BrowserWindow } from 'electron';

export function setupAutoUpdater(mainWindow: BrowserWindow) {
  autoUpdater.logger = log;
  autoUpdater.autoDownload = false;
  autoUpdater.autoInstallOnAppQuit = true;

  autoUpdater.on('checking-for-update', () => {
    mainWindow.webContents.send('updater:checking');
  });

  autoUpdater.on('update-available', (info) => {
    mainWindow.webContents.send('updater:available', info);
  });

  autoUpdater.on('update-not-available', () => {
    mainWindow.webContents.send('updater:not-available');
  });

  autoUpdater.on('download-progress', (progress) => {
    mainWindow.webContents.send('updater:progress', progress);
  });

  autoUpdater.on('update-downloaded', () => {
    mainWindow.webContents.send('updater:downloaded');
  });

  autoUpdater.on('error', (error) => {
    mainWindow.webContents.send('updater:error', error.message);
  });

  setTimeout(() => {
    autoUpdater.checkForUpdates();
  }, 5000);
}
```

### Example 5: Native Menu Setup

```typescript
// main/menu.ts
import { app, BrowserWindow, Menu, shell } from 'electron';

export function createMenu(mainWindow: BrowserWindow) {
  const isMac = process.platform === 'darwin';

  const template: Electron.MenuItemConstructorOptions[] = [
    ...(isMac
      ? [
          {
            label: app.name,
            submenu: [
              { role: 'about' },
              { type: 'separator' },
              { role: 'services' },
              { type: 'separator' },
              { role: 'hide' },
              { role: 'hideOthers' },
              { role: 'unhide' },
              { type: 'separator' },
              { role: 'quit' }
            ]
          }
        ]
      : []),
    {
      label: 'File',
      submenu: [
        {
          label: 'Open File',
          accelerator: 'CmdOrCtrl+O',
          click: () => mainWindow.webContents.send('menu:open-file')
        },
        {
          label: 'Save',
          accelerator: 'CmdOrCtrl+S',
          click: () => mainWindow.webContents.send('menu:save')
        },
        { type: 'separator' },
        isMac ? { role: 'close' } : { role: 'quit' }
      ]
    },
    {
      label: 'Help',
      submenu: [
        {
          label: 'Documentation',
          click: () => shell.openExternal('https://kit.svelte.dev')
        }
      ]
    }
  ];

  Menu.setApplicationMenu(Menu.buildFromTemplate(template));
}
```

## Anti-Patterns

### Don't: Enable nodeIntegration

```typescript
// DANGEROUS - never do this
const win = new BrowserWindow({
  webPreferences: {
    nodeIntegration: true,
    contextIsolation: false
  }
});

// Safe default
const secureWin = new BrowserWindow({
  webPreferences: {
    preload: path.join(__dirname, 'preload.js'),
    contextIsolation: true,
    nodeIntegration: false,
    sandbox: true
  }
});
```

### Don't: Access Node APIs Directly in SvelteKit Routes

```svelte
<!-- BAD: renderer code should not import node:fs -->
<script>
  import { readFileSync } from 'node:fs';
</script>

<!-- GOOD: call IPC exposed by preload -->
<script>
  async function openFile() {
    const selectedPath = await window.electron.invoke('dialog:open-file', {});
    if (!selectedPath) return;

    const result = await window.electron.invoke('file:read', { path: selectedPath });
    console.log(result);
  }
</script>
```

### Don't: Keep SSR Enabled for Packaged Electron Renderer

```typescript
// BAD for file:// packaged renderer
export const ssr = true;

// GOOD for desktop renderer
export const ssr = false;
export const prerender = true;
```

## Quick Reference

| Task | Pattern |
|------|---------|
| Create renderer app | `pnpm dlx sv create renderer` |
| Build renderer for Electron | `@sveltejs/adapter-static` + `ssr = false` |
| Main process file access | Use Node.js `fs` module in main |
| Renderer file access | IPC through preload |
| Shared renderer state | `.svelte.ts` modules with runes |
| Persistent storage | `electron-store` in main process |
| Auto-updates | `electron-updater` |
| System tray | `Tray` class in main |
| Keyboard shortcuts | `globalShortcut.register()` |
| Code signing | `electron-builder` config |

## Resources

- [Electron Documentation](https://www.electronjs.org/docs/latest)
- [SvelteKit Documentation](https://kit.svelte.dev/docs)
- [Svelte 5 Runes](https://svelte.dev/docs/svelte/what-are-runes)
- [electron-builder](https://www.electron.build/)
- [Electron Security Checklist](https://www.electronjs.org/docs/latest/tutorial/security)
