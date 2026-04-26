# pdfcrop — Portable Windows Desktop App

A portable Windows 11 desktop application for cropping PDF files, powered by:

- **[pdfcrop](https://github.com/pdfcrop/pdfcrop)** — Rust library compiled to WebAssembly for fast, private PDF processing
- **[PDF.js](https://mozilla.github.io/pdf.js/)** — PDF rendering in the embedded browser
- **[Neutralinojs](https://neutralino.js.org/)** — Lightweight desktop app framework (no Electron overhead)
- **TypeScript + Tailwind CSS + Vite** — Modern frontend stack

All processing happens **100% locally** — your PDF never leaves your device.

---

## Features

- 🗂️ **Drag & Drop** — Drop a PDF directly onto the app window
- ✂️ **Auto-detect** — Automatically detect content boundaries using rendering
- 🎯 **Manual Selection** — Draw custom crop regions per page
- 📏 **Flexible Margins** — Adjust margins (uniform or per-side)
- 📑 **Page Range** — Crop all, odd, even, or custom page ranges
- 💾 **Native Save Dialog** — Save cropped PDF using the Windows native file picker
- ⚡ **Fast** — Powered by Rust + WebAssembly
- 🔒 **100% Private** — No network required, no data uploaded

---

## Windows 11 Requirements

- Windows 10 21H2 or Windows 11 (WebView2 pre-installed)
- No additional software required for the portable app

---

## Download & Run (Pre-built)

1. Go to the **[Actions](../../actions)** tab → select the latest successful **Build Windows Portable App** run
2. Under **Artifacts**, download `pdfcrop-windows-portable.zip`
3. Extract the ZIP to any folder (e.g., `C:\Tools\pdfcrop\`)
4. Double-click `pdfcrop.exe` to launch

> **Note**: Windows may show a SmartScreen warning for unsigned executables. Click **"More info" → "Run anyway"** to proceed. The app is open-source and safe.

---

## Build Locally on Windows 11

### Prerequisites

- [Node.js 20+](https://nodejs.org/) (includes npm)
- [Rust](https://rustup.rs/) — with `wasm32-unknown-unknown` target
- [wasm-pack](https://rustwasm.github.io/wasm-pack/installer/)
- Git

### Step-by-Step

```powershell
# 1. Clone this repository
git clone https://github.com/citricguy/pdfcrop-neutralino-windows.git
cd pdfcrop-neutralino-windows

# 2. Clone the pdfcrop Rust library (needed to build WASM)
git clone https://github.com/pdfcrop/pdfcrop.git pdfcrop-rust

# 3. Add the wasm32 target to Rust
rustup target add wasm32-unknown-unknown

# 4. Install wasm-pack (if not already installed)
cargo install wasm-pack

# 5. Build the WASM module (outputs to pkg/ in this project)
cd pdfcrop-rust
wasm-pack build --target web --release --out-dir ../pkg
cd ..

# 6. Install npm dependencies
npm install

# 7. Build the web app
npm run build

# 8. Download Neutralinojs binaries
npm run neu:update

# 9. Build the desktop app
npm run neu:build
```

The Windows executable will be at:
```
dist\pdfcrop\pdfcrop-win_x64.exe
dist\pdfcrop\resources.neu
dist\pdfcrop\WebView2Loader.dll  (if present)
```

Copy those three files to any folder and run `pdfcrop-win_x64.exe`.

### All-in-one build command

```powershell
npm run dist
```

---

## Development Mode

```powershell
# Build WASM first (required once, or when Rust source changes)
cd pdfcrop-rust
wasm-pack build --target web --release --out-dir ../pkg
cd ..

# Install dependencies
npm install

# Run Vite dev server
npm run dev
```

Visit `http://localhost:8080` in your browser. Note: in dev mode, the Neutralino save dialog is not available — the app will use browser download instead.

For full desktop dev mode with Neutralino:
```powershell
npm run neu:update
npx neu run
```

---

## Testing the App

### Test Drag & Drop

1. Open File Explorer and navigate to any PDF file
2. Drag the PDF file from File Explorer onto the app window
3. The PDF should load and display in the viewer

### Test File Browse

1. Click **"Choose File"** or click anywhere in the drop zone
2. Select a PDF file from the file dialog
3. The PDF should load and display

### Test Crop & Save

1. Load a PDF file
2. Optionally: Click **"Detect Crop Region"** for auto-detection, OR draw a region manually on the PDF canvas
3. Adjust margins using the sliders if needed
4. Select page range (All Pages / Current Page / Custom)
5. Click **"Crop & Save PDF"**
6. The native Windows save dialog will appear — choose where to save
7. The cropped PDF is saved to the selected location

### If Something Goes Wrong

1. Press **F12** to open DevTools in the Neutralino window
2. Check the **Console** tab for error messages
3. Report issues at [GitHub Issues](../../issues) with:
   - Screenshot of the error
   - Console output
   - Windows version
   - PDF file info (approximate size, page count)

---

## How It Works

1. **PDF Loading**: PDF.js renders your PDF in a canvas element inside the Neutralinojs WebView2 webview
2. **Crop Region**: Draw a selection rectangle over the canvas, or use auto-detect (calls the Rust WASM module)
3. **Cropping**: The Rust `pdfcrop` library (compiled to WASM) processes the PDF bytes in-memory
4. **Saving**: In desktop mode, `Neutralino.os.showSaveDialog()` opens a native file picker; `Neutralino.filesystem.writeBinaryFile()` writes the cropped PDF

---

## Project Structure

```
pdfcrop-neutralino-windows/
├── .github/
│   └── workflows/
│       └── build.yml          # CI: build WASM + web app + Neutralino Windows exe
├── src/
│   └── js/
│       ├── app.ts             # Main app (Neutralino save dialog integration)
│       ├── bbox-overlay.ts    # Interactive crop region drawing
│       ├── pdf-viewer.ts      # PDF.js rendering wrapper
│       ├── utils.js           # Utility functions
│       └── worker.js          # Web worker placeholder
├── pkg/                       # WASM output (generated, not committed)
├── resources/                 # Vite build output (generated, not committed)
├── dist/                      # Neutralinojs app output (generated, not committed)
├── index.html                 # Main HTML with Neutralino.js script tag
├── input.css                  # Tailwind CSS + Inter font import
├── neutralino.config.json     # Neutralinojs configuration
├── package.json               # npm config and build scripts
├── vite.config.ts             # Vite config (outputs to resources/)
├── tsconfig.json              # TypeScript config
├── tailwind.config.js         # Tailwind CSS config
└── postcss.config.js          # PostCSS config
```

---

## CI/CD

GitHub Actions automatically builds the Windows portable app on every push to `main`:

1. Checks out this repo + the `pdfcrop/pdfcrop` Rust library
2. Installs Rust + `wasm32-unknown-unknown` target
3. Builds the WASM module with `wasm-pack`
4. Runs `npm ci` + `npm run build` (Vite)
5. Runs `neu update` + `neu build --release` (Neutralinojs)
6. Uploads `pdfcrop-windows-portable.zip` as a build artifact

---

## License

MIT OR Apache-2.0

Based on [pdfcrop](https://github.com/pdfcrop/pdfcrop) by [Wuqiong Zhao](https://wqzhao.org).
