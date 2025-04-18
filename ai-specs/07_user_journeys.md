## User Experience Flows

### Onboarding & Room Entry:

1.  **User opens the app/website.**
2.  **User logs in using Firebase Authentication** (e.g., Google Sign-In).
3.  **App initializes Socket.io connection and authenticates.**
4.  **User sees a list of available/upcoming tournament rooms** (fetched via REST or Socket). List shows room name, players (x/y), time remaining/duration.
5.  **User clicks "Join" on a desired room.**
6.  **App displays a loading/joining indicator.**
7.  **On successful `roomJoined` event:**
    * User is navigated to the main game screen for that room.
    * Screen shows the Slot Machine component, the real-time Leaderboard component, and the Room Timer.
    * Initial leaderboard and time are displayed.
8.  **On error** (`room_full`, `already_ended`, etc.): Display error message to the user.

### Gameplay Loop (Inside a Room):

1.  **User sees the Slot Machine ready to play.**
2.  **User sees the leaderboard** (showing own rank/score and others).
3.  **User sees the countdown timer for the room.**
4.  **User clicks the "GO!" (Spin) button.**
    * Button might briefly disable during spin animation.
5.  **Client emits `spinRequest`.**
6.  **Slot machine reels animate** (based on existing `slotMachine.vue` logic, potentially triggered *after* receiving server confirmation if latency is high, or immediately for perceived responsiveness).
7.  **Client receives `spinResult` from the server.**
8.  **Reel animation stops** on the `result` symbols provided by the server.
9.  **If `coinsWon > 0`:**
    * Display winning animation/message.
    * Update the player's displayed score (based on `newTotalCoins`).
10. **Client receives `leaderboardUpdate` from the server.**
11. **Leaderboard component updates smoothly** to reflect new scores and rankings.
12. **Room timer display updates.**
13. **User repeats the spin process.**

### Room End & Results:

1.  **User observes the timer counting down.**
2.  **Timer reaches 00:00.**
3.  **Spin button becomes disabled.**
4.  **Client receives `roomEnded` event.**
5.  **App displays a "Room Finished!" message.**
6.  **App highlights the winner(s)** based on the payload.
7.  **App displays the `finalLeaderboard`.**
8.  **Provide options to "Find New Game" or "Go Home".**
9.  **User is removed from the room's socket channel.**

### Leaving / Disconnecting:

* **Intentional Leave:**
    1.  User clicks a "Leave Room" button.
    2.  App emits `leaveRoom`.
    3.  App cleans up local state (Vuex `room.js`).
    4.  App navigates user away from the game screen.
* **Disconnection:**
    1.  Network drops.
    2.  Socket.io client attempts reconnection.
    3.  **On Reconnect:**
        * App re-authenticates.
        * Potentially needs logic to `checkSession` or rejoin the *same* room if the server supports it and the room is still active.
        * If rejoin supported, server sends current `roomJoined` state.
        * If not, user is effectively out of that game.
    4.  **If Reconnect Fails / Room Ended:** User is presented with an error or navigated back to the room list/home screen.
