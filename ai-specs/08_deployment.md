### Frontend Deployment (Vue.js)

* **Build:** Use Vue CLI (`npm run build`) to generate optimized static assets (HTML, CSS, JS).
* **Hosting:** Firebase Hosting is recommended.
    * Configure `firebase.json` to point to the `dist` directory.
    * Setup rewrites for single-page application (SPA) routing:

    ```json
    // firebase.json
    {
      "hosting": {
        "public": "dist",
        "ignore": [
          "firebase.json",
          "**/.*",
          "**/node_modules/**"
        ],
        "rewrites": [
          {
            "source": "**",
            "destination": "/index.html"
          }
        ]
      }
    }
    ```

* **CDN:** Firebase Hosting automatically uses a global CDN for assets.
* **PWA/Service Worker:**
    * Configure Vue CLI's PWA plugin (`@vue/cli-plugin-pwa`) to generate a `manifest.json` and a service worker (`service-worker.js`).
    * The service worker should cache application shell assets (HTML, JS, CSS, main images) for faster loading and basic offline capability (showing the app frame even if offline). It won't enable gameplay offline.

### Backend Deployment (Node.js / Socket.io)

* **Environment:** Node.js LTS version.
* **Dependency Management:** Use `package.json` and `npm install`.
* **Process Management:** Use a process manager like `PM2` for:
    * Keeping the server running (restarts on crash).
    * Running the server in cluster mode (`pm2 start app.js -i max`) to utilize multiple CPU cores. **Crucial for Socket.io scalability, requires adapter like `socket.io-redis`.**
    * Log management.
* **Hosting Options:**
    * **Google Cloud Run:** Serverless container platform. Scales automatically (including to zero). Good for variable loads. Need containerization (`Dockerfile`).
    * **Google App Engine (Standard/Flex):** PaaS. Handles scaling and infrastructure. Standard requires specific Node.js runtimes; Flex uses containers.
    * **Google Compute Engine:** IaaS. Full control over VMs. Requires manual setup of Node.js, PM2, NGINX, firewall, etc.
    * **Other Cloud Providers:** AWS Elastic Beanstalk, Azure App Service, DigitalOcean App Platform, Heroku offer similar PaaS/Serverless options.
* **Reverse Proxy (NGINX):** Recommended, especially if not using a PaaS that handles it.
    * Place NGINX in front of the Node.js application.
    * Handles SSL/TLS termination.
    * Can serve static assets if needed (though frontend is likely separate).
    * Configured to proxy WebSocket connections (`proxy_pass`, `proxy_http_version 1.1`, `proxy_set_header Upgrade`, `proxy_set_header Connection "upgrade"`).
    * Can provide basic load balancing if running multiple Node instances on the same VM.
* **Scalability Considerations:**
    * **Vertical Scaling:** Increase CPU/RAM of the server instance(s). Limited.
    * **Horizontal Scaling:** Run multiple instances of the Node.js server behind a load balancer (like NGINX or cloud provider's LB).
    * **Socket.io Scaling:** When running multiple instances, Socket.io needs an **Adapter** (e.g., Redis Adapter - `socket.io-redis`) so that broadcasts from one instance reach clients connected to other instances. This is critical for `leaderboardUpdate`, `userJoined`, etc. to work correctly across a cluster.
* **Environment Variables:** Use environment variables for configuration (Database credentials, Firebase Admin SDK config, JWT secrets, CORS origins, Redis connection string if used, etc.). **Do not hardcode secrets.**
