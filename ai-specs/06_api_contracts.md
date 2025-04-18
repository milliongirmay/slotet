### REST API Endpoints (Minimal)

* **(Optional)**
    * **Description:** Fetches a list of currently available (waiting, active) or upcoming (scheduled) rooms. Could alternatively be handled via an initial Socket.io request/response.
    * **Response Body:** `[{ roomId, name, status, currentPlayerCount, maxPlayers, durationSeconds, entryFee }]`

### Socket.io API

* **Namespace:** `/` (Default)

* **Client -> Server Events:**

    * **authenticate**
        * **Payload:** `{ token: <FirebaseIdTokenString> }`
        * **Description:** Sent immediately after connection to authenticate the user.
        * **Server Response:** `authenticated` or `auth_error` event.

    * **joinRoom**
        * **Payload:** `{ roomId: <String> }`
        * **Description:** Request to join a specific room.
        * **Server Response:** `roomJoined` (to sender), `userJoined` (broadcast), `leaderboardUpdate` (broadcast), or `error` (to sender).

    * **spinRequest**
        * **Payload:** `{ roomId: <String> }`
        * **Description:** User requests to spin the reels in their current room.
        * **Server Response:** `spinResult` (to sender), `leaderboardUpdate` (broadcast), or `error` (to sender).

    * **leaveRoom**
        * **Payload:** `{ roomId: <String> }`
        * **Description:** User intentionally leaves the room.
        * **Server Response:** Server cleans up, broadcasts `userLeft` and `leaderboardUpdate`. No direct response needed to sender usually.

    * **(Optional) requestRoomList**
        * **Payload:** `{}` (None)
        * **Description:** Alternative to REST API for getting the initial room list.
        * **Server Response:** `roomListUpdate` event.

* **Server -> Client Events:**

    * **authenticated**
        * **Payload:** `{ userId: <String>, displayName: <String> }`
        * **Description:** Confirms successful authentication.

    * **auth\_error**
        * **Payload:** `{ message: <String> }`
        * **Description:** Sent if token verification fails.

    * **roomJoined**
        * **Payload:** `{ roomId: <String>, roomState: { status: <String>, players: <Array>, timeLeftSeconds: <Number> }, leaderboard: <Array> }`
        * **Description:** Sent to the client that successfully joined a room. Contains initial state.
        * **Example `players`:** `[{ userId, displayName, currentCoins }, ...]`
        * **Example `leaderboard`:** `[{ userId, displayName, currentCoins }, ...]` (sorted)

    * **userJoined**
        * **Payload:** `{ userId: <String>, displayName: <String>, usersInSession: <Number> }`
        * **Description:** Broadcast to existing users in a room when a new user joins.

    * **userLeft**
        * **Payload:** `{ userId: <String>, usersInSession: <Number> }`
        * **Description:** Broadcast to remaining users in a room when a user leaves or disconnects.

    * **spinResult**
        * **Payload:** `{ result: <Array<String>>, coinsWon: <Number>, newTotalCoins: <Number> }`
        * **Description:** Sent to the client who initiated the spin, confirming the outcome.
        * **Example `result`:** `['cherry', 'seven', 'bar']`

    * **leaderboardUpdate**
        * **Payload:** `{ leaderboard: <Array>, timeLeftSeconds: <Number> }`
        * **Description:** Broadcast to all users in a room periodically and/or after score changes.
        * **Example `leaderboard`:** `[{ userId, displayName, currentCoins, rank: <Number> }, ...]` (sorted)

    * **roomEnded**
        * **Payload:** `{ winner: { userId: <String>, displayName: <String> } | null, finalLeaderboard: <Array> }`
        * **Description:** Broadcast to all users in a room when the timer expires. `winner` might be null if no one scored.

    * **error**
        * **Payload:** `{ message: <String>, type: <String> (e.g., 'join_failed', 'spin_failed', 'room_full', 'not_in_room') }`
        * **Description:** General error reporting to a specific client.

    * **(Optional) roomListUpdate**
        * **Payload:** `[{ roomId, name, status, currentPlayerCount, maxPlayers, durationSeconds, entryFee }]`
        * **Description:** Sends the list of available rooms.
