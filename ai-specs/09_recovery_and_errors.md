## Error Handling & Recovery

### Client-Side:

* Use `try...catch` blocks for async operations (API calls, Vuex actions).
* Display user-friendly error messages (e.g., using a notification system). Avoid showing raw technical errors.
* Listen for generic `error` events from the Socket.io server and specific error events (`auth_error`, `join_failed`, etc.). Update UI accordingly.
* Implement loading states and disable buttons during operations to prevent duplicate requests.

### Server-Side:

* Use `try...catch` in asynchronous handlers (DB operations, external API calls).
* Validate all incoming data from clients (payload structure, data types, permissions).
* Emit specific `error` events back to the relevant client on failures (e.g., `socket.emit('error', { message: 'Room is full', type: 'join_failed' })`).
* Implement robust logging (e.g., using Winston or Pino) to track errors and debug issues. Log relevant context (userId, roomId).
* Gracefully handle unexpected errors to prevent server crashes (using global error handlers and PM2 for restarts).

### Recovery Mechanisms

#### Socket.io Disconnection/Reconnection:

* **Client:** Socket.io client library automatically attempts reconnection on temporary network issues.
* **Client:** On successful `connect` event after a disconnect, the client **must** re-authenticate by emitting the `authenticate` event with the Firebase token.
* **Server:** Maintain user session information associated with `userId`, not just `socket.id`. When a user re-authenticates with the same `userId`, check if they were previously in an `active` room.

#### Rejoining Room (Server Logic):

* If the user's previous room (`roomId`) is still `active` and the server supports rejoin:
    * Re-associate the new `socket.id` with the user in the room's `players` map.
    * Mark the player as `isConnected: true`.
    * Emit `roomJoined` to the rejoining client with the **current** `roomState` and `leaderboard`.
    * Broadcast `userRejoined` (or similar) and `leaderboardUpdate` to others.
* If the room has ended or rejoin is not supported, treat as a fresh connection.

#### Client State Sync:

* The `roomJoined` event sent upon successful join/rejoin is the primary mechanism for syncing the client's state with the server's current state for that room.

#### Leaderboard Synchronization:

* The server is the single source of truth for the leaderboard.
* On joining/rejoining a room, the client receives the full current leaderboard via the `roomJoined` event.
* During gameplay, the client relies on `leaderboardUpdate` broadcasts from the server to stay synchronized. No client-side calculation of rankings should occur.

#### Offline Handling (Frontend):

* The PWA service worker can cache the application shell, allowing the app to load partially even when offline.
* Display a clear "Offline" indicator if the Socket.io connection fails and cannot reconnect.
* Disable gameplay features (Spin button, room joining) when offline.
* Gameplay itself **requires** an active connection.

### Potential Error States & Handling

* **Room Full:** Server receives `joinRoom`, checks player count `>= maxPlayers`. Emits `error({ message: 'Room is full', type: 'join_failed' })` to the requesting client. Client UI shows message.
* **Invalid/Ended Room:** Server receives `joinRoom` for invalid/ended `roomId`. Emits `error({ message: 'Room not found or has ended', type: 'join_failed' })` to client. Client UI shows message.
* **Authentication Failure:** Server fails to verify Firebase token on `authenticate`. Emits `auth_error`. Client UI prompts for re-login.
* **Room Not Active (Spin):** Server receives `spinRequest` but room is not `active`. Emits `error({ message: 'Room is not active', type: 'spin_failed' })` to client. Client UI might show "Waiting for game to start" or disable spin button.
* **Insufficient Funds (Spin - if implemented):** If spins cost from a global balance, server checks balance. Emits `error({ message: 'Insufficient funds', type: 'spin_failed' })`. Client UI shows message.
* **Unexpected Server Error:** Unexpected error during spin calculation, DB write, etc. Server logs detailed error. Emits generic `error({ message: 'An unexpected error occurred. Please try again.', type: 'server_error' })` to client.
