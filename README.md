# sobylon

A small multiplayer 3D arena prototype: an Express + Socket.IO room server and a Babylon.js + cannon.js browser client where players roll spheres around an OBJ map and try to knock each other off.

**Status:** Archived prototype (October–December 2023). Lobby, room settings, map selection, physics movement, and last-player-standing game end work; the TODO items below were never implemented and the project is no longer developed.

## Features

- Lobby: create a room with a name, map, and max player count; join by nickname and texture; live room list broadcast to all clients
- Room owner can tune world settings (gravity, speed, jump height/cooldown, dash power/cooldown, damping, restitution, max life) before starting
- Seven OBJ maps (`bridge`, `spintower`, `thefield`, `blackhole`, `space`, `donut`, `untitled`) with per-map spawn points
- Physics-driven sphere movement with jump (`Space`) and dash (`ShiftLeft`) cooldowns; on-screen touch joystick for mobile
- Position/velocity relay between clients; falling below the map costs a life and the last player alive wins
- Ownership hand-off when the room owner disconnects; server-side colored event log retrievable via a `log` event

## How it works

- `back/src/index.ts` runs Express on port 3000, serves `front/dist` under `/public`, and keeps an in-memory `worlds` array. Socket.IO handlers: `createRoom`, `joinRoom`, `getRooms`, `updateRoom`, `startGame`, `init`, `update`, `gameOver`, `debug`, `log`, `disconnect`. There is no database; all state lives in the process.
- `front/src/index.ts` is a single-file Babylon.js client: it renders the lobby from `index.html` templates, loads `obj/<map>.obj` with `babylonjs-loaders`, creates a cannon.js physics impostor for the local sphere, and streams its position/velocity to the server every frame while mirroring remote players as named spheres.
- `front/src/textures.ts` builds PBR materials from the texture sets in `front/dist/texture`.
- Shared `types.d.ts` files in `back/src` and `front/src` define `World` and `Player`.
- `vercel.json` routes every request to `back/dist/index.js` for deployment; the compiled `back/dist` and `front/dist` are committed for that reason.

## Tech stack

- Runtime/Language: Node.js, TypeScript
- Server: Express 4, Socket.IO 4
- Client: Babylon.js 6, babylonjs-loaders, babylonjs-gui, cannon 0.6, socket.io-client 4
- Bundling: webpack 5 (`front/webpack.config.js`), ts-loader, html-webpack-plugin
- Infra/Tooling: Vercel (`vercel.json`), nodemon + concurrently for server dev

## Getting started

### Requirements

- Node.js 18+ and npm

### Install

```bash
cd back && npm install && cd ..
cd front && npm install && cd ..
```

### Run

Server (compiles `back/src` to `back/dist` and restarts on change):

```bash
cd back
npm run dev        # concurrently "tsc -w" "nodemon ./dist/index.js"
# or, using the committed build:
npm start          # node ./dist/index.js
```

Client bundle (output to `front/dist`):

```bash
cd front
npx webpack        # uses webpack.config.js; webpack-dev-server is also installed
```

Open `http://localhost:3000/` — it redirects to `/public/index.html`.

## Project structure

```
back/
  src/index.ts     Express + Socket.IO room server (in-memory worlds)
  src/types.d.ts   World / Player types
  dist/            compiled server (committed for Vercel)
front/
  src/index.ts     Babylon.js client: lobby, scene, physics, networking
  src/index.html   lobby and in-game HTML/CSS
  src/textures.ts  PBR material helpers
  dist/            webpack output, OBJ maps, textures (committed)
  webpack.config.js
vercel.json        routes all requests to back/dist/index.js
package.json       root start script (node back/dist/index.js)
```

## Configuration

No environment variables are read. The server port (3000) and the client socket URL (`io('/')`, same origin) are hard-coded.

## Known gaps (original TODO list)

Room exit/lobby exit buttons, lobby profile UI, game-over transition, kill system, chat system, room name UI, room passwords and public/private rooms, database-backed accounts, ranked/public/custom matchmaking, and a rank dashboard were planned but not built.

## License

No license file yet (`package.json` declares ISC, but the repository contains no LICENSE file).
