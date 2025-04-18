## Atomic Tasks for AI Agents

These tasks should be executable independently or sequentially by Firebase Studio agents.

### Phase 1: Backend Setup & Core Room Logic

* **Task:** Initialize Node.js project structure, install dependencies (`socket.io`, `firebase-admin`, `@socket.io/redis-adapter` or similar if clustering planned).
* **Task:** Adapt existing Ludoet Socket.io server setup: configure server, basic connection handling.
* **Task:** Implement Firebase Admin SDK initialization and JWT verification middleware for Socket.io connections (`authenticate` event handler).
* **Task:** Implement basic in-memory store for active rooms (`Map` or `Object`).
* **Task:** Implement `joinRoom` event handler: validate room, add user to room (in-memory), join socket to room channel, emit `roomJoined`, broadcast `userJoined`. (Initially, ignore max players, status checks).
* **Task:** Implement `leaveRoom` event handler: remove user from room (in-memory), leave socket channel, broadcast `userLeft`.
* **Task:** Implement `disconnect` event handler: perform leave room logic.
* **Task:** Define Room `status` logic (`waiting`, `active`, `ended`). Add `maxPlayers` check to `joinRoom`.
* **Task:** Implement basic room timer logic: Start timer when room becomes `active`, broadcast `roomEnded` when timer expires. Store `timeLeftSeconds` in active room state.
* **Task:** Implement server-side data structures for `slots.js` and `paytable.js` (mirroring frontend).
* **Task:** Implement `spinRequest` handler: **Basic** - generate random placeholder result, emit `spinResult` to sender (no scoring yet). Ensure it only works in `active` rooms.

### Phase 2: Scoring, Leaderboard & Frontend Integration

* **Task:** Implement server-side spin result calculation based on reel configuration.
* **Task:** Implement server-side paytable checking and `coinsWon` calculation based on spin result.
* **Task:** Update `spinRequest` handler: Calculate `coinsWon`, update player's `currentCoins` in the active room state, include `coinsWon` and `newTotalCoins` in `spinResult` event.
* **Task:** Implement server-side leaderboard generation (sorting players by `currentCoins`).
* **Task:** Implement `leaderboardUpdate` broadcast: Send sorted leaderboard and `timeLeftSeconds` to all clients in the room (triggered after score changes and periodically by timer).
* **Task:** (Vue) Create `auth.js` Vuex module. Integrate Firebase Auth SDK for login/logout.
* **Task:** (Vue) Create `socket.js` Vuex module. Implement `initializeSocket` action (connect, handle `connect`/`disconnect`, emit `authenticate`).
* **Task:** (Vue) Create `room.js` Vuex module (state, mutations for basic room data: `roomId`, `status`, `players`, `timeLeftSeconds`).
* **Task:** (Vue) Modify App structure: Add Room List view, Game Room view. Implement basic routing.
* **Task:** (Vue) Connect `socket.js` to `auth.js` to get token for `authenticate`. Handle `authenticated`/`auth_error` events.
* **Task:** (Vue) Implement UI for displaying room list (fetch via placeholder/socket).
* **Task:** (Vue) Implement "Join Room" button functionality: Dispatch action using `socket.js` to emit `joinRoom`.
* **Task:** (Vue) Implement `socket.js` listeners for `roomJoined`, `userJoined`, `userLeft`. Dispatch actions to update `room.js` Vuex state.
* **Task:** (Vue) Display basic room info (players, timer) in the Game Room view using data from `room.js` Vuex module.
* **Task:** (Vue) Adapt `slotMachine.vue`: Trigger spin animation. Connect "Spin" button to `socket.js` action emitting `spinRequest`. Remove client-side balance deduction on spin.
* **Task:** (Vue) Implement `socket.js` listener for `spinResult`. Dispatch action/mutation to update player's score display (or trigger leaderboard update). Remove client-side reward calculation (`changeBallance`).
* **Task:** (Vue) Implement Leaderboard component: Display data from `room.js` Vuex `leaderboard` state.
* **Task:** (Vue) Implement `socket.js` listener for `leaderboardUpdate`. Dispatch mutation to update `room.js` Vuex `leaderboard` and `timeLeftSeconds` state. Ensure leaderboard component updates reactively.
* **Task:** (Vue) Implement `socket.js` listener for `roomEnded`. Update `room.js` state, display winner and final leaderboard. Provide navigation options.

### Phase 3: Persistence, Polish & Deployment Prep

* **Task:** Implement Winner selection logic on the server when room ends. Include winner in `roomEnded` payload.
* **(Optional) Task:** Define Firestore/RTDB schemas (`Users`, `Rooms`).
* **(Optional) Task:** Implement server logic to persist final leaderboard and winner to the `Rooms` collection in DB when `roomEnded`.
* **(Optional) Task:** Implement server logic to fetch/update user profile (`Users` collection) on auth/events.
* **Task:** Implement robust error handling on server (`try-catch`, specific error events).
* **(Vue) Task:** Implement UI display for server errors received via socket `error` events.
* **(Vue) Task:** Implement client-side reconnection logic (re-authenticate on `connect`).
* **Task:** Configure PM2 for backend process management (`ecosystem.config.js`). If clustering (`-i max`), add necessary Socket.io adapter (`@socket.io/pm2` or Redis).
* **Task:** Create Dockerfile for backend service (if deploying via Cloud Run, App Engine Flex, etc.).
* **Task:** Configure Firebase Hosting for frontend (`firebase.json`).
* **(Vue) Task:** Configure PWA plugin (`vue.config.js` or similar) and test service worker caching.
* **Task:** Implement security measures: Input validation on server, rate limiting (basic).
