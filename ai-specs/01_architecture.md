### System Architecture Overview

The system follows a client-server model optimized for real-time interactions.
+---------------------+ +------------------------+ +-----------------------+
| Vue.js Frontend |----->| Node.js/Socket.io BE |<---->| Firebase Firestore/RTDB |
| (Browser/PWA) |<-----| (Game Logic, Realtime) | | (User Data, Room Meta)|
| - UI Components | | - Room Management | +-----------------------+
| - State (Vuex) | | - Spin Logic (Server) | ^
| - Socket.io Client | | - Leaderboard Mgmt | |
| - Firebase Auth SDK | | - Timer Logic | v
+---------------------+ +------------------------+ +-----------------------+
^ | Firebase Auth |
| | (Authentication) |
+------------------------+-----------------------+
### Component Responsibilities

1.  **Vue.js Frontend:**
    *   Renders the game interface (slot machine, leaderboard, room UI).
    *   Manages local UI state and interacts with Vuex for application state.
    *   Initiates connection and communicates with the Socket.io backend via events.
    *   Handles user authentication flow using the Firebase Auth SDK.
    *   Displays real-time data received from the backend (leaderboard, scores, time).

2.  **Node.js / Socket.io Backend:**
    *   **Core Adaptation:** Modify the existing Ludoet Socket.io server.
    *   Manages WebSocket connections from clients.
    *   Authenticates users based on Firebase Auth tokens received over socket connection.
    *   Handles room creation, joining, and leaving logic.
    *   Maintains the state of active rooms (players, scores, timer) primarily *in memory* for performance.
    *   **Crucially: Executes the slot spin logic server-side** to determine results and rewards, preventing client-side cheating.
    *   Calculates and updates player scores within rooms.
    *   Manages real-time leaderboards for each active room.
    *   Broadcasts updates (leaderboard changes, player joins/leaves, game events) to relevant clients within rooms.
    *   Manages room timers and triggers end-of-room logic (winner calculation, result broadcast).
    *   (Optional) Persists final room results and potentially key game events to Firestore/RTDB.

3.  **Firebase Authentication:**
    *   Provides secure user sign-up and login mechanisms (Email/Password, Google, etc.).
    *   Issues ID tokens used by the client to authenticate with the backend.

4.  **Firebase Firestore / Realtime Database:**
    *   Stores persistent user profile information (linked to Firebase Auth UID).
    *   Stores metadata about available or upcoming tournament rooms (e.g., schedule, rules, entry requirements - if any).
    *   Stores final tournament results and potentially historical data.
    *   *Note:* Active, real-time game state (current scores, exact player list in a running room) is primarily held in the Node.js server's memory for low latency. The database is for persistence, not the primary source during active gameplay.

### Real-time Stack

*   **Socket.io:** Used for bidirectional communication between the Vue.js frontend and the Node.js backend. Essential for leaderboard updates, score changes, and room events. The existing Ludoet server's connection handling, room broadcasting, and session management concepts will be adapted.
