# Media Studio

**Media Studio** is a robust, Electron-based desktop application designed for seamless media management and viewing. It allows users to browse local directories, visualize media collections (images and videos) in a responsive grid layout, and perform quick actions like zooming, filtering, sorting, and searching.

Built with performance and user experience in mind, Media Studio leverages native system capabilities to generate video thumbnails on the fly and integrates smoothly with the default OS media viewer.

---

## 🚀 Features

- **Folder Browsing**: Easily select and load local directories to view their contents.
- **Rich Media Grid**:
  - Displays high-quality previews for images (`.png`, `.jpg`, `.jpeg`, `.webp`, `.gif`).
  - **Auto-generated Video Thumbnails**: Uses FFmpeg to generate thumbnail previews for video files (`.mp4`, `.mov`, `.avi`, `.mkv`).
- **Dynamic Zooming**: Adjust the grid item size dynamically (50% to 250%) using UI controls.
- **Advanced Filtering & Sorting**:
  - Filter content by type (Images only, Videos only, or All).
  - Sort by Name (A-Z, Z-A) or by File Type.
- **Instant Search**: Real-time filtering of files by name.
- **System Integration**: Double-click any file to open it in the default system media viewer.
- **Persistent State**: Automatically remembers the last opened folder for a seamless restart experience.
- **Responsive Design**: Modern, dark-themed UI with a clean toolbar and status bar.

---

## 🛠 Tech Stack

- **Core Framework**: [Electron](https://www.electronjs.org/) (v30.0.0)
- **Frontend**: HTML5, Vanilla CSS3 (Custom Variables, Flexbox/Grid), JavaScript (ES Modules).
- **Backend/System**: Node.js (File System API, Child Process).
- **Media Processing**: [FFmpeg](https://ffmpeg.org/) (spawned process for thumbnail generation).
- **Build System**: [electron-builder](https://www.electron.build/) (v24.13.3).

---

## 📦 Installation

To check out and run this project locally, you will need **Node.js** installed on your machine.

### Prerequisites
- **Node.js**: [Download here](https://nodejs.org/).
- **FFmpeg**: The application requires `ffmpeg` to be available in your system's PATH to generate video thumbnails.
  - **Windows**: [Download and add to PATH](https://ffmpeg.org/download.html).

### Steps

1. **Clone the repository** (if applicable) or navigate to the project directory:
   ```bash
   cd media-viewer
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

---

## ▶️ How to Run

### Development Mode
To start the application in development mode with hot-reloading (if configured) or standard debug logging:

```bash
npm start
```

### Production Build
To create a distributable installer for your operating system (Windows .exe by default):

```bash
npm run dist
```
The output files (installers) will be generated in the `dist` folder.

---

## 🏗 Architecture Overview

The application follows the standard **Electron Main/Renderer** architecture pattern, enforcing security best practices like Context and Process Isolation.

### 1. Main Process (`src/main/main.js`)
- **Responsibility**: Manages the application lifecycle, creates browser windows, and handles native system operations.
- **Key Modules**:
  - `fs`: Direct file system access to read directories and manage settings.
  - `child_process`: Spawns `ffmpeg` processes to generate video thumbnails.
  - `ipcMain`: Listens for asynchronous events from the renderer.
- **Security**: nodeIntegration is disabled; interaction happens solely via IPC.

### 2. Preload Script (`src/preload/preload.js`)
- **Responsibility**: Acts as a safe bridge between the Main and Renderer worlds.
- **Mechanism**: Uses `contextBridge` to expose a limited, type-safe API (`window.studio`) to the frontend.

### 3. Renderer Process (`src/renderer/renderer.js`)
- **Responsibility**: Handles all UI rendering, user interactions, and DOM manipulations.
- **Logic**: 
  - Manages application state (`currentFolder`, `filters`, `sortOrder`).
  - Implements caching for thumbnails (`thumbCache`) to prevent redundant processing.
  - Uses strictly standard Web APIs (DOM) for maximum performance and partial rendering techniques (lazy loading images).

---

## 🔌 API Endpoints (IPC Channels)

Since this is a desktop app, "API" refers to the Internal IPC Channels exposed via `window.studio`.

| Channel | Method | Description |
| :--- | :--- | :--- |
| `select-folder` | `invoke` | Opens the native OS dialog to select a folder. Returns the path. |
| `get-last-folder` | `invoke` | Retrieves the path of the last opened folder from persistent settings. |
| `get-media-files` | `invoke` | Input: `folderPath`, `allowedExts`. Returns a list of absolute file paths. |
| `generate-thumbnail` | `invoke` | Input: `videoPath`. Generates a `.jpg` thumbnail via FFmpeg and returns its path. |
| `open-file` | `invoke` | Input: `filePath`. Opens the file using the OS default application. |

---

## 🔒 Security

- **Context Isolation**: Enabled (`contextIsolation: true`). The renderer process has no direct access to Node.js primitives or the `require` function.
- **Sandboxing**: Enabled (`sandbox: true`).
- **Content Security Policy (CSP)**: Applied via `<meta>` tag in `index.html` to restrict script sources to `'self'` and image sources to local files, preventing XSS and remote injection attacks.

---

## ⚡ Performance & Scaling

- **Thumbnail Caching**: Generated thumbnails are saved to the file system (in user data) and cached in memory (Map) during the session to ensure immediate rendering on subsequent loads.
- **Lazy Loading**: Images use `loading="lazy"` to reduce initial memory footprint and load time for folders with thousands of images.
- **Asynchronous Operations**: Heavy I/O operations (file scanning, ffmpeg generation) are handled asynchronously in the Main process to keep the UI (Renderer) responsive.

---

## 📝 Notes

- **Settings Persistence**: User preferences (like the last opened folder) are stored in a simple JSON file in the OS `UserData` directory.
- **FFmpeg Dependency**: Ensure `ffmpeg` is strictly installed on the host machine. For a production release, it is recommended to bundle the `ffmpeg` binary within the app resources and point the spawn command to that internal path.
