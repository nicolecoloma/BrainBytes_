# BrainBytes

Project Overview

BrainBytes is an AI-driven tutoring platform created to make quality academic support more accessible to Filipino students. The project is built using modern DevOps methodologies and containerization technologies to ensure scalability, reliability, and efficient deployment.

Team Member

Nicole Andrea Coloma
Email: lr.nacoloma@mmdc.mcl.edu.ph

Project Goals

Develop a containerized application with a well-configured networking setup.
Build an automated CI/CD pipeline using GitHub Actions to streamline development and deployment.
Deploy the application on Railway.app for reliable cloud hosting.
Implement monitoring and observability solutions to track application performance and system health.

Technology Stack

| **Layer**        | **Technology**       |
| ---------------- | -------------------- |
| Frontend         | Next.js              |
| Backend          | Node.js              |
| Database         | MongoDB Atlas        |
| Containerization | Docker               |
| CI/CD            | GitHub Actions       |
| Cloud Provider   | Railway.app          |
| Monitoring       | Prometheus & Grafana |

## CI/CD Workflow

BrainBytes uses **GitHub Actions** to automate code validation, build verification, security scanning, and deployment preparation throughout the development lifecycle.

### Workflow Purposes

| **Workflow**   | **Purpose**                                                                                                                                                  |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `quality.yml`  | Detects linting issues early using ESLint and generates annotations and JSON reports for both the frontend and backend.                                      |
| `ci.yml`       | Executes frontend and backend test suites and collects code coverage reports to validate application functionality.                                          |
| `build.yml`    | Verifies that Docker images build successfully and confirms that the Docker Compose stack can start and stop without errors.                                 |
| `deploy.yml`   | Prepares the application for deployment and records deployment metadata. The actual deployment step is currently implemented as a placeholder.               |
| `security.yml` | Performs security checks on dependencies and container images using **npm audit**, **Snyk**, and **Trivy** to identify known vulnerabilities.                |
| `main.yml`     | Orchestrates the complete CI/CD pipeline, including linting, testing, build verification, deployment preparation, security scanning, and end-to-end testing. |

## Manual Workflow Execution

Several GitHub Actions workflows support manual execution through the `workflow_dispatch` trigger.

### Workflows Available for Manual Runs

| **Workflow**   | **Manual Execution** |
| -------------- | -------------------- |
| `main.yml`     | ✅ Supported          |
| `build.yml`    | ✅ Supported          |
| `deploy.yml`   | ✅ Supported          |
| `security.yml` | ✅ Supported          |

### How to Run a Workflow Manually

1. Open your GitHub repository.
2. Navigate to the **Actions** tab.
3. Select the workflow you want to run.
4. Click **Run workflow**.
5. Choose the branch to execute the workflow on.
6. Click **Run workflow** to start the process.

### Automatic Workflow Execution

The following workflows do **not** include the `workflow_dispatch` trigger and are executed automatically when code is pushed or a pull request is opened:

| **Workflow**  | **Trigger**            |
| ------------- | ---------------------- |
| `quality.yml` | `push`, `pull_request` |
| `ci.yml`      | `push`, `pull_request` |

If an automatically triggered workflow fails, you can rerun it by opening the workflow run page in the **Actions** tab and selecting **Re-run jobs**.

## Troubleshooting

If a workflow fails, begin by reviewing the job logs in **GitHub Actions**. Then, reproduce the issue locally by running the corresponding commands in the affected application directory.

| **Issue**                                 | **Recommended Solution**                                                                                                                                                                         |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Linting failures**                      | Run `npm ci` followed by `npm run lint:js` in both `brainbytes-multi-container/frontend` and `brainbytes-multi-container/backend`.                                                               |
| **Test failures**                         | Execute `npm test` in the appropriate frontend or backend folder to reproduce the issue before reviewing the workflow logs.                                                                      |
| **Docker build or Docker Compose errors** | Ensure Docker Desktop is running and verify that the Docker Compose configuration matches the service paths referenced in the GitHub Actions workflows.                                          |
| **Security scan failures**                | Confirm that the `SNYK_TOKEN` repository secret is configured correctly and that the image names used by the Trivy scan match those generated during the build process.                          |
| **Deployment health check failure**       | Keep in mind that the current deployment workflow uses a placeholder deployment script, so a live service may not yet be available for health checks.                                            |
| **Unable to run a workflow manually**     | Verify that the workflow includes the `workflow_dispatch` trigger. Otherwise, it can only be started by a `push`, `pull_request`, or by selecting **Re-run jobs** from an existing workflow run. |



