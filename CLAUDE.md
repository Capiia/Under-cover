# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Undercover is a web-based multiplayer party game for Capgemini frog seminars (Happiness Team). Players receive secret words and must identify the "undercover" agents through discussion and voting. The app runs entirely client-side with Firebase Realtime Database for synchronization.

## Architecture

Single-page app in one file (`index.html`, ~1500 lines) containing HTML + CSS + JS. No build step, no bundler, no framework.

- **Firebase Realtime Database** handles all multiplayer sync (rooms, players, game state)
- **Three roles**: Host (H object), Classic host (C object), Player (P object)
- **Game flow**: Create room → Players join via QR/code → Assign roles (civilian/undercover/mr.white) → Discussion rounds → Vote → Eliminate → Win condition check
- **`words.json`**: Array of word pairs `{civ, spy, cat}` used for gameplay
- **State is stored in Firebase** at `/rooms/{4-letter-code}/` with real-time listeners (`hListen`, `cListen`, `pListen`)
- **Host authentication**: `hostToken` (crypto.randomUUID) stored in Firebase, verified server-side via Security Rules

## Hosting Constraints


**Current distribution**: Files are on SharePoint (frogUndercover team site). SharePoint/OneDrive blocks JavaScript execution in HTML files. The Embed web part also blocks `<script>` tags and `srcdoc` iframes with JS.

**Viable paths forward** (without IT):
- Power Apps via make.powerapps.com (user has access + SharePoint has PowerApps web part)
- SharePoint page with instructions to open file locally ("Ouvrir dans l'application")

## Firebase Security Rules

The Realtime Database rules properly secure the app:
- `hostToken` is **immutable** (`.write: false`) — cannot be changed after room creation
- Game state fields (status, round, etc.) require matching hostToken for writes
- Players can only write to their own `connected`, `vote`, `ready`, `mwGuess` fields with type validation
- Room creation requires `!data.exists()` and valid hostToken

## Security Notes

- Firebase config in `index.html` is public by design (client-side Firebase app) — security is enforced by the rules above
- `esc()` function (line ~1484) sanitizes user input for innerHTML via textContent
- `escAttr()` function handles JS+HTML escaping for inline event handlers
- `genToken()` uses `crypto.randomUUID()` (cryptographically secure)
- `rnd()` uses `Math.random()` — acceptable for game logic (role shuffling), not for security

## Key Functions

- `hgCreate()` / `csCreate()` — create a room (host mode / classic mode)
- `csStart()` — assign roles and start game
- `renderGroup()` / `cRenderGame()` — render host game UI based on status
- `pRoute()` — route player to correct screen based on game status
- `checkWin()` — evaluate win conditions after elimination
- `cleanupExpiredRooms()` — auto-delete rooms older than 24h
