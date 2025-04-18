## Security Considerations & Measures

### Authentication:

* All Socket.io connections **must** be authenticated using Firebase Auth tokens (`authenticate` event).
* Server **must** verify the token using Firebase Admin SDK before trusting the `userId`.
* Associate the verified `userId` with the `socket.id` for the duration of the connection. Reject messages from unauthenticated sockets.

### Authorization:

* Server must verify that the authenticated `userId` is actually part of the `roomId` specified in incoming events (`spinRequest`, `leaveRoom`). Prevent users from affecting rooms they are not in.

### Input Validation (Server-Side):

* Validate the structure and type of all incoming socket event payloads (e.g., `roomId` should be a string).
* Sanitize any user-generated content if displayed (e.g., display names) to prevent XSS (though Firebase display names are generally safe).

### Trusted Game Logic (CRITICAL):

* Spin results and coin rewards **must be determined server-side.**
* The client **only** sends a `spinRequest`. It **never** sends its perceived result or winnings.
* The server sends the authoritative `spinResult` (symbols, `coinsWon`, `newTotalCoins`) back to the client.
* The client's role is purely presentation based on server data.

### Rate Limiting:

* Implement server-side rate limiting on sensitive actions, especially `spinRequest`.
* Limit requests per user per unit of time (e.g., max 1 spin request every 1-2 seconds) to prevent spamming/abuse/potential exploits. Use an in-memory store (like a `Map` with timestamps) or Redis for tracking.

### Secure Communication:

* Use HTTPS for the frontend hosting.
* Use WSS (WebSocket Secure) for the Socket.io connection. This is typically handled by configuring SSL/TLS on the reverse proxy (NGINX) or the PaaS hosting environment.

### Dependency Management:

* Keep server and client dependencies updated (`npm audit`) to patch known vulnerabilities.

### Environment Variables:

* Store sensitive configuration (Firebase Admin SDK credentials, API keys, secrets) in environment variables, not in the codebase.

### Denial of Service (DoS) Mitigation:

* Basic rate limiting helps.
* Cloud hosting platforms often provide infrastructure-level DoS protection.
* Ensure efficient server logic to prevent resource exhaustion from valid traffic.
