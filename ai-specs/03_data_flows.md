### Real-time Data Flows

1.  **User Authentication & Socket Connection:**
    *   `Client (Vue)`: User logs in via Firebase Auth SDK. Gets Firebase ID Token.
    *   `Client (Vue)`: Initiates Socket.io connection.
    *   `Client (Vue)`: On `connect` event, emits `authenticate` event with `{ token: firebaseIdToken }`.
    *   `Server (Node)`: Receives `authenticate`. Verifies token with Firebase Admin SDK.
    *   `Server (Node)`: Stores `userId` associated with the `socket.id`.
    *   `Server (Node)`: Emits `authenticated` or `auth_error` back to the client.

2.  **Finding & Joining a Room:**
    *   `Client (Vue)`: (After auth) Fetches available/waiting rooms (e.g., via REST or initial socket event `requestRoomList`).
    *   `Server (Node)`: Sends `roomListUpdate` with rooms having `status == 'waiting'` or `status == 'active'`.
    *   `Client (Vue)`: User selects a room (`roomId`). Emits `joinRoom` with `{ roomId }`.
    *   `Server (Node)`: Receives `joinRoom`.
        *   Validates `roomId`.
        *   Checks if room exists and is joinable (`waiting` or `active` and not full).
        *   Checks authentication (`userId` associated with `socket.id`).
        *   Adds user to the room's `players` list in server memory (initial `currentCoins = 0`).
        *   Associates `socket.id` with the room (`socket.join(roomId)`).
        *   Updates `leaderboard` in server memory.
        *   Emits `roomJoined` *to the joining client* with `{ roomId, roomState: { status, players, timeLeftSeconds }, leaderboard }`.
        *   Broadcasts `userJoined` *to all other clients in the room* with `{ userId, displayName, usersInSession: count }`.
        *   Broadcasts `leaderboardUpdate` *to all clients in the room* with `{ leaderboard, timeLeftSeconds }`.
        *   If joining starts the game (e.g., first player or min players reached), update room `status` to `active`, start the timer, set `actualStartTime` and `endTime`.

3.  **Player Spin:**
    *   `Client (Vue)`: User clicks "Spin". Checks if enough "room balance" (conceptual, as spins might be free or cost 1 room coin). Emits `spinRequest` with `{ roomId }`. *(Note: Starter code deducts from a general balance; this needs adaptation to room context or free spins)*.
    *   `Server (Node)`: Receives `spinRequest`.
        *   Validates user is authenticated and in the specified `roomId`.
        *   Checks if the room is `active`.
        *   **Performs server-side spin logic:** Generates random reel results based on configured probabilities/symbols.
        *   Calculates winnings based on the result and the game's paytable.
        *   Updates the requesting user's `currentCoins` in the room's `players` map.
        *   Updates the `leaderboard` in server memory.
        *   Emits `spinResult` *back to the requesting client* with `{ result: reelSymbols, coinsWon: amount, newTotalCoins: user.currentCoins }`.
        *   Broadcasts `leaderboardUpdate` *to all clients in the room* with `{ leaderboard, timeLeftSeconds }`.

4.  **Leaderboard & Timer Updates:**
    *   `Server (Node)`:
        *   On every score change (after a winning spin).
        *   Periodically (e.g., every 1-5 seconds) via a `setInterval` loop associated with the active room.
        *   Decrements `timeLeftSeconds`.
        *   Broadcasts `leaderboardUpdate` *to all clients in the room* with `{ leaderboard, timeLeftSeconds }`.

5.  **Room Ends:**
    *   `Server (Node)`: Room timer reaches zero (`timeLeftSeconds <= 0`).
        *   Stops the timer interval.
        *   Sets room `status` to `ending`.
        *   Determines the winner(s) based on the final `leaderboard`.
        *   Broadcasts `roomEnded` *to all clients in the room* with `{ winner: { userId, displayName }, finalLeaderboard }`.
        *   Updates room `status` to `ended`.
        *   Persists `finalLeaderboard` and `winnerUserId` to the Room document in the DB.
        *   Cleans up the active room state from server memory after a short delay or confirmation.
        *   Forces disconnection or moves clients out of the ended room socket channel.

6.  **User Disconnects / Leaves:**
    *   `Server (Node)`: Detects `disconnect` event for a `socket.id`.
        *   Finds the room the socket was in.
        *   Marks the player as `isConnected: false` in the room's `players` map (or removes them if rejoin isn't supported).
        *   Broadcasts `userLeft` *to all other clients in the room* with `{ userId }`.
        *   Broadcasts `leaderboardUpdate` (as the disconnected player might be removed or just marked).
    *   `Client (Vue)`: User explicitly clicks "Leave". Emits `leaveRoom` with `{ roomId }`.
    *   `Server (Node)`: Receives `leaveRoom`, performs similar cleanup as disconnect.
