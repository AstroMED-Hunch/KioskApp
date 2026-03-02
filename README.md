# KioskApp

A real-time interactive kiosk application for managing inventory boxes with face recognition, WebSocket communication, and a modern cyberpunk-inspired UI.

## Project Overview

KioskApp is a Svelte-based kiosk interface that enables users to:
- Register and manage inventory boxes
- Check boxes in and out of storage shelves
- Receive real-time updates via WebSocket connections
- Experience face recognition-based authentication
- View current shelf inventory status with a sleek, futuristic design

The application communicates with backend services for box registration, shelf management, and face recognition tasks.

## Tech Stack

- **Frontend Framework**: Svelte 5 with SvelteKit
- **Build Tool**: Vite
- **Styling**: Augmented UI (cyberpunk UI effects)
- **Real-time Communication**: WebSocket
- **Package Manager**: npm

## Prerequisites

Before you begin, ensure you have the following installed:
- Node.js (v18 or higher)
- npm (comes with Node.js)

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd KioskApp
```

2. Install dependencies:
```bash
npm install
```

## Setup

The application requires two backend services to be running:
- **WebSocket Server** on `ws://localhost:3000` - Handles real-time box and shelf operations
- **REST API Server** on `http://localhost:8000` - Provides shelf information via `/getShelves` endpoint

Make sure these services are running before starting the kiosk application.

## Running the Application

### Development Mode

Start the development server with hot module reloading:
```bash
npm run dev
```

The application will be available at `http://localhost:5173` (or another port if 5173 is in use).

To automatically open the app in your browser:
```bash
npm run dev -- --open
```

### Production Build

Create an optimized production build:
```bash
npm run build
```

Preview the production build locally:
```bash
npm run preview
```

## Project Structure

```
src/
├── routes/
│   ├── +layout.svelte      # Main layout wrapper
│   └── +page.svelte        # Main kiosk interface
├── lib/
│   ├── Popup.svelte        # Reusable popup modal component
│   ├── ShelfEntry.svelte   # Shelf display component
│   ├── index.js            # Library exports
│   └── assets/
│       └── favicon.ico      # App icon
├── app.css                 # Global styles and theme variables
└── app.html                # HTML entry point
```

## Features

- **Real-time Box Management**: Register boxes and receive instant confirmation
- **Face Recognition**: Integrated face detection for user authentication during transactions
- **Shelf Tracking**: View and manage box locations across multiple shelves
- **WebSocket Integration**: Live updates from backend services
- **Responsive UI**: Modern cyberpunk aesthetic with augmented UI styling
- **Modal Popups**: User-friendly popup notifications for interactions

## Styling

The application uses a custom color scheme defined in `src/app.css`:
- Primary Color: `#66fcf1` (Cyan)
- Secondary Color: `#45a29e` (Teal)
- Background: `#0b0c10` (Near-black)
- Panel: `#1f2833` (Dark gray)

The design leverages the [Augmented UI](https://augmented-ui.com/) library for cyberpunk-style corners and effects.

## Troubleshooting

- **Connection errors**: Ensure backend services are running on the correct ports
- **WebSocket failures**: Check that the WebSocket server is accessible at `ws://localhost:3000`
- **Missing shelves data**: Verify the REST API is responding to `/getShelves` requests
- **Build issues**: Clear `node_modules` and reinstall with `npm install`