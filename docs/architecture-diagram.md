# BrainBytes Deployment Architecture

## System Architecture Overview

```text
                         BrainBytes Deployment Architecture

                           +----------------------+
                           |    User's Browser    |
                           +----------+-----------+
                                      |
                                   HTTPS
                                      |
                                      v
                      +-------------------------------+
                      | Railway Frontend (Next.js)    |
                      +---------------+---------------+
                                      |
                        HTTP (Internal Railway Network)
                                      |
                                      v
                      +-------------------------------+
                      | Railway Backend (Express.js)  |
                      +---------------+---------------+
                                      |
                   +------------------+------------------+
                   |                                     |
          MongoDB Wire Protocol                     HTTPS
                   |                                     |
                   v                                     v
      +---------------------------+         +----------------------+
      | Railway MongoDB Database  |         |     Groq AI API      |
      +---------------------------+         +----------------------+

      SSL/HTTPS: Railway Managed
      Internal Communication: Railway Internal DNS
      Database: Railway Managed MongoDB Plugin
```

## Service Communication

| **From** | **To**      | **Protocol**          | **Communication Method**                                                        |
| -------- | ----------- | --------------------- | ------------------------------------------------------------------------------- |
| Browser  | Frontend    | HTTPS                 | Railway automatically provides the application URL with SSL/TLS encryption.     |
| Frontend | Backend     | HTTP                  | Uses the Railway internal service URL configured through environment variables. |
| Backend  | MongoDB     | MongoDB Wire Protocol | Connects using the Railway-managed MongoDB connection string.                   |
| Backend  | Groq AI API | HTTPS                 | Sends secure external API requests over HTTPS.                                  |

## Security Layers

| **Layer**              | **Protection**                                                                 |
| ---------------------- | ------------------------------------------------------------------------------ |
| **Transport**          | SSL/TLS encryption automatically managed by Railway.                           |
| **Secrets Management** | Environment variables are securely encrypted and managed by Railway.           |
| **Network Security**   | Internal service networking prevents direct public access to the database.     |
| **API Security**       | Express routes perform input validation before processing requests.            |
| **CORS**               | Cross-Origin Resource Sharing is restricted to the production frontend domain. |
