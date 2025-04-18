## External Service Integrations

### Firebase Authentication:

* **Client:** Use Firebase JS SDK (`firebase/auth`) for UI flows (login, signup, logout). Obtain the ID token (`user.getIdToken()`).
* **Server:** Use Firebase Admin SDK (`firebase-admin`) to verify the ID token received via the `authenticate` socket event (`admin.auth().verifyIdToken(token)`). Extract `uid` (`userId`).
* **Configuration:** Initialize Firebase SDKs on both client and server with project credentials. Server uses a Service Account key (stored securely, e.g., env var).

### Firebase Firestore / Realtime Database (Optional but Recommended):

* **Server:** Use Firebase Admin SDK (`firebase-admin/firestore` or `firebase-admin/database`) to interact with the database.
* **Usage:** Store user profiles, persistent room metadata (schedule, past results), potentially audit logs.
* **Security Rules:** Configure Firestore/RTDB security rules to restrict direct client access if necessary, enforcing data access primarily through the backend server. For this design, most DB access is server-side.

### CDN (Content Delivery Network):

* **Frontend:** Firebase Hosting automatically provides a CDN for static assets (JS, CSS, images). Ensure assets are configured for appropriate caching headers.
* **Game Assets:** Store slot machine images, sounds, etc., on the CDN (Firebase Hosting works well) for fast loading globally. Reference them via relative paths or CDN URLs.

### Socket.io Adapter (Required for Scaling):

* **Server:** If deploying multiple Node.js instances (clustering/horizontal scaling), an adapter is mandatory for broadcasting events across instances.
* **Options:**
    * `@socket.io/redis-adapter`: Requires a Redis instance. Robust and common choice.
    * `@socket.io/mongo-adapter`: Requires MongoDB.
    * `@socket.io/postgres-adapter`: Requires PostgreSQL.
    * `@socket.io/pm2`: Helper for use with PM2 cluster mode, simpler for single-machine clustering.
* **Integration:** Install the adapter (`npm install @socket.io/redis-adapter redis`) and configure Socket.io to use it:

```javascript
// Server setup
const { Server } = require("socket.io");
const { createAdapter } = require("@socket.io/redis-adapter");
const { createClient } = require("redis");
const pubClient = createClient({ url: "redis://localhost:6379" }); // Use env var for URL
const subClient = pubClient.duplicate();

Promise.all([pubClient.connect(), subClient.connect()]).then(() => {
  const io = new Server({ adapter: createAdapter(pubClient, subClient) });
  // ... rest of socket server setup ...
  io.listen(3000); // Or your port
});
