# Randomizer (Vue + Vite)

A lightweight browser-only app to randomly select items from any list. Add multiple independent panels, paste items (one per line), and pick at random. All state is saved to your browser's localStorage.

## Features
- Multiple randomizer panels
- Paste items (one per line) into each panel
- Randomize, clear, duplicate, and remove panels
- Auto-save panel state (text and last choice) to localStorage
- Zero backend; deploy as static files

## Getting started

### Prerequisites
- Node.js 18+ (recommended)

### Install and run
```bash
npm install
npm run dev
```
Then open the URL shown in the terminal (usually http://localhost:5173).

### Build for production
```bash
npm run build
npm run preview
```

## Usage tips
- Each newline represents a separate item.
- Use Duplicate to create templates you can reuse.
- Your lists are stored locally only on your device.

## Tech stack
- Vue 3 (script setup, Composition API)
- Vite

## License
MIT

