### Project Summary

Develop a real-time, multiplayer Slot Machine game where users join time-limited tournament rooms, spin slots to earn coins, and compete on a live leaderboard. The player with the most coins when the room timer expires wins.

### Platform Goals

*   **Engaging Multiplayer Experience:** Foster competition and social interaction through tournament rooms and live leaderboards.
*   **Real-time Interaction:** Provide immediate feedback on spins, coin gains, and leaderboard changes.
*   **Scalability:** Design the backend to handle numerous concurrent rooms and players.
*   **Robustness:** Ensure reliable gameplay, score tracking, and recovery from potential disconnections.
*   **AI-Driven Development:** Utilize Firebase Studio agents for efficient implementation based on this specification.
*   **Leverage Existing Infrastructure:** Adapt the existing Ludoet Socket.io server for room management and real-time updates.

### User Types

*   **Player:** Authenticated user who can join rooms, play the slot machine, view leaderboards, and see results.
*   **System/Admin (Future):** Potential role for managing rooms, settings, or monitoring. (Not in initial scope).
*   **AI Developer Agent (Firebase Studio):** Consumes this specification to implement features and fixes.

### Core Technologies

*   **Frontend:** Vue.js (v2/v3 as appropriate for starter code), Vuex, Vue Router
*   **Backend:** Node.js, Socket.io (adapting existing Ludoet server)
*   **Authentication:** Firebase Authentication
*   **Database (Potential):** Firebase Firestore or Realtime Database (for user profiles, room metadata, final results). Server memory for active game state.
*   **Deployment:** Node.js hosting environment (e.g., Google Cloud Run, App Engine, Compute Engine), Firebase Hosting (for frontend).
