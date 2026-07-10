# BrainBytes CI/CD Setup

## Overview

BrainBytes uses **GitHub Actions** to automate its Continuous Integration and Continuous Deployment (CI/CD) process. The primary workflow is located at `.github/workflows/main.yml` and is configured to run on pushes to the `main`, `initial`, and `mj-automation` branches, on pull requests, and through scheduled weekly executions.

## Pipeline Workflows

| **Stage**                              | **Purpose**                                                                      | **Trigger**                                             | **Key Features**                                                                                                                       |
| -------------------------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Lint (`lint`)**                   | Performs code quality checks using ESLint on both frontend and backend projects. | Pushes and Pull Requests                                | Runs on Node.js 20.x and 22.x, generates ESLint reports, and provides inline pull request annotations.                                 |
| **2. Test (`test`)**                   | Executes Jest unit and integration tests with code coverage.                     | After successful linting                                | Uses a MongoDB 4.4 service container and uploads frontend/backend coverage reports as workflow artifacts.                              |
| **3. Build (`build`)**                 | Builds Docker images for the frontend and backend applications.                  | After successful tests                                  | Validates Docker Compose configuration and stores built images as workflow artifacts.                                                  |
| **4. Security Scan (`security-scan`)** | Detects dependency and container vulnerabilities.                                | After successful build                                  | Uses **npm audit**, **Snyk**, and **Trivy**. Requires the `SNYK_TOKEN` repository secret.                                              |
| **5. End-to-End Tests (`e2e`)**        | Runs Playwright browser tests against the full application stack.                | After successful build                                  | Starts the application with Docker Compose, executes Playwright tests, and automatically retries failed tests.                         |
| **6. Deploy (`deploy`)**               | Deploys the application to the Railway.app environment.                          | After successful build and E2E tests (push events only) | Deploys `main` to production and `mj-automation` to the test environment. Railway automatically performs deployment and health checks. |

## Manual Workflow Execution

To run the CI/CD workflow manually:

1. Open the GitHub repository.
2. Navigate to the **Actions** tab.
3. Select the **BrainBytes CI/CD** workflow.
4. Click **Run workflow**.
5. Choose the desired branch.
6. Click **Run workflow** to start the pipeline.

## Workflow Status

| **Stage**      | **Status Badge** |
| -------------- | ---------------- |
| CI/CD Pipeline | `CI/CD`          |

## Required Repository Secrets

Configure the following secrets under **Settings → Secrets and variables → Actions**.

| **Secret**      | **Purpose**                                                   |
| --------------- | ------------------------------------------------------------- |
| `SNYK_TOKEN`    | Authenticates Snyk for dependency and vulnerability scanning. |
| `GROQ_API_KEY`  | Provides access to the Groq AI service during local testing.  |
| `RAILWAY_TOKEN` | Enables automated deployments to Railway.app.                 |

## Troubleshooting

| **Issue**                                           | **Solution**                                                                                                                                                                                                               |
| --------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ESLint reports are empty**                        | Run `npm run lint:js` locally to verify the ESLint configuration.                                                                                                                                                          |
| **End-to-end tests fail with "Connection Refused"** | Ensure Docker Compose successfully starts all services, verify port **7000** is mapped correctly, and confirm the frontend serves HTTP on port **3000**.                                                                   |
| **Security scan failures**                          | For **Snyk 401** errors, generate a new API token and update the `SNYK_TOKEN` secret. If **Trivy** fails, ensure Docker images have been built before scanning.                                                            |
| **Slow pipeline builds**                            | Verify that GitHub Actions dependency and Docker layer caching are producing cache hits by reviewing the workflow logs.                                                                                                    |
| **Railway deployment fails**                        | Confirm that the `RAILWAY_TOKEN` is valid, review Railway build logs, verify the application runs locally with `docker compose up`, and ensure all required environment variables are configured in the Railway dashboard. |
