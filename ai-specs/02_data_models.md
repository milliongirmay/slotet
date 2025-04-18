### Database/Storage Schema

*(Note: 'DB' refers to Firestore/RTDB for persistent data. 'Server Memory' refers to data held by the Node.js/Socket.io server during active gameplay)*.

1.  **Users Collection (DB - Firestore recommended)**
    *   `userId` (String - Primary Key, same as Firebase Auth UID)
    *   `displayName` (String)
    *   `email` (String - Optional, from Auth)
    *   `photoURL` (String - Optional, from Auth)
    *   `globalStats` (Object - Optional, e.g., total winnings, games played)
        *   `totalCoinsWon` (Number)
        *   `tournamentsPlayed` (Number)
        *   `tournamentsWon` (Number)
    *   `createdAt` (Timestamp)
    *   `lastLogin` (Timestamp)

2.  **Rooms Collection (DB - Firestore/RTDB)**
    *   `roomId` (String - Primary Key, Auto-generated)
    *   `name` (String - e.g., "Hourly Slot Blitz")
    *   `status` (String Enum: `scheduled`, `waiting`, `active`, `ended`, `archived`)
    *   `scheduledTime` (Timestamp - Optional, for future rooms)
    *   `actualStartTime` (Timestamp - Set when room becomes `active`)
    *   `endTime` (Timestamp - Calculated on start: `actualStartTime` + `duration`)
    *   `durationSeconds` (Number - e.g., 300 for 5 minutes)
    *   `maxPlayers` (Number)
    *   `entryFee` (Number - Optional, defaults to 0)
    *   `prizeStructure` (Object/String - Defines winner payouts)
    *   `finalLeaderboard` (Array<Object> - Stored when room ends)
        *   `{ userId: String, displayName: String, finalCoins: Number, rank: Number }`
    *   `winnerUserId` (String - Optional, stored when room ends)

3.  **Active Room State (Server Memory)**
    *   Managed per `roomId`.
    *   `roomId` (String)
    *   `status` (String Enum: `waiting`, `active`, `ending`)
    *   `players` (Map<String, Object> - Keyed by `userId`)
        *   `userId` (String)
        *   `displayName` (String)
        *   `socketId` (String)
        *   `currentCoins` (Number - Coins earned *within this room*)
        *   `isConnected` (Boolean)
    *   `leaderboard` (Array<Object> - Sorted snapshot of players by `currentCoins`)
        *   `{ userId: String, displayName: String, currentCoins: Number }`
    *   `timer` (Object - Holds interval/timeout reference)
    *   `timeLeftSeconds` (Number - Decremented server-side)

4.  **Spin Log (Optional - DB or Logging System)**
    *   Could be useful for auditing/debugging, but potentially high volume.
    *   `spinId` (String - Auto-generated)
    *   `userId` (String)
    *   `roomId` (String)
    *   `timestamp` (Timestamp)
    *   `betAmount` (Number - Initially fixed, e.g., 1)
    *   `spinResult` (Array<String> - Symbols on each reel)
    *   `coinsWon` (Number)
    *   `userCoinTotalAfterSpin` (Number - User's coin total *in the room*)
