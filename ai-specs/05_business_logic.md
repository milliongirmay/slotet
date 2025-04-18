## Core Game Logic

### Slot Machine Mechanics:

* **Reel Configuration:** Defined by `store/modules/slots.js` data (e.g., `items: ["bar2", "bar3", "cherry", "bar", "seven"]`). This configuration **must also exist securely on the server.**
* **Spin Result Generation (Server-Side):**
    * When a `spinRequest` is received, the server generates a random result for each reel independently.
    * Probabilities can be simple (equal chance for each symbol) or weighted (more complex, defined server-side).
    * The server determines the final stopping position for each reel (e.g., the symbol aligned with the middle payline).
* **Paytable Logic (Server-Side):**
    * Defined by `store/modules/paytable.js` data (**mirrored on the server**).
    * After generating the `spinResult`, the server compares the symbols against the paytable rules (line wins, scatter wins, combinations).
    * It calculates the total `coinsWon` for that spin based **only** on the server-side result and paytable.
* **Reward Mechanics:**
    * Coins are awarded **only** based on the `coinsWon` calculated by the server during the `spinRequest` processing.
    * The server updates the player's `currentCoins` for the room in its memory.
    * The `spinResult` event sent to the client includes the `coinsWon` and the `newTotalCoins`, which the client uses to update the display and Vuex state (via mutations).

### Tournament Room Lifecycle:

* **Waiting:** Room exists, players can join. Timer is not running.
* **Active:** Minimum players joined (or first player joins if configured), or `scheduledTime` reached.
    * Server sets `actualStartTime`, `endTime`.
    * Server starts the room timer (`setInterval` decrementing `timeLeftSeconds`).
    * Server broadcasts initial `leaderboardUpdate`.
    * Players can spin and earn coins. New players might be allowed to join (up to `maxPlayers`) until a certain point or for the whole duration.
* **Ending:** `timeLeftSeconds` reaches 0.
    * Server stops accepting `spinRequest` events for this room.
    * Server calculates the winner(s).
* **Ended:** Winner announced, final results broadcast.
    * Server persists final state to DB.
    * Server cleans up room from memory. Clients are disconnected from the room channel.

### Winner Selection:

* When the room timer ends (status becomes `ending`), the server identifies the player(s) with the highest `currentCoins` in the final `leaderboard`.
* **Tie Handling:** Define a clear rule:
    * **Option 1 (Simple):** All tied players are declared winners (prize might be split if applicable).
    * **Option 2:** First player to reach the winning score wins (requires tracking score timestamps - adds complexity).
    * **Default:** Implement Option 1 unless specified otherwise.
* The winner information is included in the `roomEnded` broadcast.

### Coin Model:

* Coins earned (`currentCoins`) are specific to the tournament room instance. They reset to 0 when joining a new room.
* Spins within the room are currently assumed to be free (no cost deducted). If spins should cost something, it must deduct from the `room's` `currentCoins` or a `global` balance (requires `balance.js` and server-side deduction logic). **Default assumption: Spins are free within the tournament.**
