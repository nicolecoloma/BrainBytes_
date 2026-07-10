# BrainBytes Cloud Environment Setup (Railway.app)

## Overview

BrainBytes is deployed on **Railway.app** using the free tier. Railway simplifies cloud deployment by automatically building and deploying applications from a GitHub repository while providing built-in networking, SSL/TLS, health checks, and automatic scaling.

---

## 1. Railway Project Setup

### Step 1: Create a Railway Account

1. Visit **https://railway.app**.
2. Sign in using your GitHub account.
3. Authorize Railway to access your GitHub repositories.

### Step 2: Deploy from GitHub

1. Click **New Project**.
2. Select **Deploy from GitHub Repo**.
3. Choose the **grahamcrackers123/BrainBytes** repository.
4. Configure the project services as shown below.

| **Service** | **Root Directory**                    | **Build Command**              | **Start Command** |
| ----------- | ------------------------------------- | ------------------------------ | ----------------- |
| Backend     | `brainbytes-multi-container/backend`  | `npm install`                  | `npm start`       |
| Frontend    | `brainbytes-multi-container/frontend` | `npm install && npm run build` | `npm run start`   |
| MongoDB     | Railway MongoDB Plugin                | —                              | —                 |

### Step 3: Configure Health Checks

| **Service** | **Health Check Path** | **Interval** | **Failure Threshold** |
| ----------- | --------------------- | ------------ | --------------------- |
| Backend     | `/`                   | 30 seconds   | 3 failures            |
| Frontend    | `/`                   | 30 seconds   | 3 failures            |

### Step 4: Configure Restart Policy

Set the **Restart Policy** for every service to **Always** so Railway automatically restarts containers if they stop unexpectedly.

---

# 2. Environment Variables

## Backend

| **Variable**   | **Value**                          | **Purpose**                                   |
| -------------- | ---------------------------------- | --------------------------------------------- |
| `GROQ_API_KEY` | Your API Key                       | Authenticates requests to the Groq AI service |
| `MONGO_URL`    | `<RAILWAY_MONGODB_URL>/brainbytes` | MongoDB connection string                     |
| `NODE_ENV`     | `production`                       | Enables production mode                       |
| `PORT`         | `3000`                             | Express server port                           |

> **Important:** Copy the value of `RAILWAY_MONGODB_URL` and append `/brainbytes`. Do **not** use `$(RAILWAY_MONGODB_URL)` because variable interpolation may not work correctly.

## Frontend

| **Variable**          | **Value**        | **Purpose**          |
| --------------------- | ---------------- | -------------------- |
| `NEXT_PUBLIC_API_URL` | `$(BACKEND_URL)` | Backend API endpoint |
| `PORT`                | `3000`           | Next.js server port  |

Railway automatically generates both `BACKEND_URL` and `RAILWAY_MONGODB_URL` after the services are linked.

---

# 3. Security Configuration

## Sensitive Environment Variables

* Store all secrets using Railway's encrypted environment variable system.
* Keep `.env` files out of version control using `.gitignore`.
* Only the `GROQ_API_KEY` needs to be provided manually.

## CORS Configuration

Restrict API access to the production frontend by configuring CORS.

```javascript
const cors = require('cors');

app.use(cors({
  origin: process.env.CORS_ORIGIN || '*',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```

Example environment variable:

```text
CORS_ORIGIN=https://your-app.railway.app
```

## Network Security

Railway provides:

* Automatic HTTPS/SSL certificates
* Private networking between services
* No public access to the MongoDB instance

---

# 4. Monitoring and Logging

## Railway Logging

Useful Railway CLI commands:

```bash
railway logs
railway logs --service backend
railway logs --tail
```

## Application Logging

Railway automatically captures application logs written to `stdout` and `stderr`.

Example:

```javascript
console.error("Error fetching messages:", err);
```

## Error Handling

```javascript
process.on('uncaughtException', (err) => {
  console.error('UNCAUGHT EXCEPTION:', err);
});

process.on('unhandledRejection', (reason) => {
  console.error('UNHANDLED REJECTION:', reason);
});
```

## Performance Monitoring

Railway includes:

* CPU usage
* Memory usage
* Deployment history
* Rollback support
* Response time metrics (with custom domains)

---

# 5. Deployment Configuration

## Project Structure

```text
BrainBytes/
├── brainbytes-multi-container/
│   ├── backend/
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   └── ...
│   ├── frontend/
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   └── ...
│   └── docker-compose.yml
├── .github/
│   └── workflows/
│       └── deploy-railway.yml
```

## GitHub Integration

Railway automatically deploys the application whenever changes are pushed to the configured branch.

1. Connect the GitHub repository.
2. Select the `main` branch.
3. Enable automatic deployment.

---

## Optional Nixpacks Configuration

### Backend (`backend/nixpacks.toml`)

```toml
[phases.setup]
nixPkgs = ['nodejs_22', 'curl']

[phases.install]
cmds = ['npm install']

[phases.build]
cmds = ['echo "No build step needed"']

[start]
cmd = 'npm start'
```

### Frontend (`frontend/nixpacks.toml`)

```toml
[phases.setup]
nixPkgs = ['nodejs_22']

[phases.install]
cmds = ['npm install --legacy-peer-deps']

[phases.build]
cmds = ['npm run build']

[start]
cmd = 'npm run start'
```

---

# 6. Service Configuration

## Backend

| **Setting**    | **Configuration**                    |
| -------------- | ------------------------------------ |
| Source         | `brainbytes-multi-container/backend` |
| Build          | Dockerfile or Nixpacks               |
| Port           | 3000                                 |
| Health Check   | `GET /`                              |
| Restart Policy | Always                               |

## Frontend

| **Setting**    | **Configuration**                     |
| -------------- | ------------------------------------- |
| Source         | `brainbytes-multi-container/frontend` |
| Build          | Dockerfile or Nixpacks                |
| Port           | 3000                                  |
| Health Check   | `GET /`                               |
| Restart Policy | Always                                |

## MongoDB

| **Setting**        | **Configuration** |
| ------------------ | ----------------- |
| Plugin             | MongoDB           |
| Version            | 7.0+              |
| Persistent Storage | Enabled           |
| Internal Port      | 27017             |

---

# 7. Custom Domain (Optional)

1. Open **Railway Dashboard → Settings → Custom Domain**.
2. Add your preferred domain.
3. Update your DNS CNAME record.
4. Railway automatically provisions an SSL certificate.

---

# 8. Troubleshooting

| **Issue**                          | **Solution**                                                                                                                                 |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend cannot connect to backend | Verify `MONGO_URL` is valid, remove any trailing slash from `NEXT_PUBLIC_API_URL`, and confirm the backend connects to MongoDB successfully. |
| `MongoParseError: Invalid scheme`  | Replace `MONGO_URL` with the actual value copied from `RAILWAY_MONGODB_URL`, followed by `/brainbytes`.                                      |
| `Cannot find module 'openai'`      | Rebuild the project or redeploy after ensuring dependencies are installed.                                                                   |
| `GROQ_API_KEY` missing             | Add the environment variable in the backend service settings.                                                                                |
| Railway deployment fails           | Verify the `RAILWAY_TOKEN`, inspect Railway build logs, and ensure auto-deployment from GitHub is enabled.                                   |

---

# 9. Deployment Checklist

| **Task**                               | **Status** |
| -------------------------------------- | ---------- |
| Railway project connected to GitHub    | ☐          |
| Backend deployed successfully          | ☐          |
| Frontend deployed successfully         | ☐          |
| MongoDB plugin configured              | ☐          |
| Environment variables added            | ☐          |
| Health checks enabled                  | ☐          |
| Restart policy set to **Always**       | ☐          |
| Production CORS configured             | ☐          |
| Auto-deploy enabled for `main`         | ☐          |
| Logs available in Railway dashboard    | ☐          |
| Application accessible via Railway URL | ☐          |
