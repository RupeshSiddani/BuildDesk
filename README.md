# BuildDesk

A self-hosted deployment platform for static websites. This project demonstrates an end-to-end flow for registering Git repositories, triggering containerized builds, and serving static sites through a reverse proxy.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
  - [Environment Variables](#environment-variables)
  - [API Server](#api-server)
  - [Frontend](#frontend)
  - [Build Server](#build-server)
  - [S3 Reverse Proxy](#s3-reverse-proxy)
- [Database Setup](#database-setup)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

## Overview

This platform allows you to deploy static websites by simply providing a Git repository URL. The system clones your repository, builds it inside a Docker container, uploads the output to S3, and serves the site through a custom subdomain.

Use this project as a learning resource or as a foundation for building your own deployment infrastructure.

## Features

- Register projects using Git repository URLs
- Automated build pipeline running inside Docker containers
- Real-time log streaming from build containers to the UI
- Log aggregation using Kafka and ClickHouse
- Static site hosting through an S3 reverse proxy
- WebSocket-based real-time updates

## Architecture

The system consists of four main components working together:

1. **API Server** - Handles project registration, deployment triggers, and orchestrates the build process
2. **Build Server** - A Docker container that clones repositories, runs builds, and uploads artifacts to S3
3. **Frontend** - A Next.js application providing the user interface for managing projects and viewing logs
4. **S3 Reverse Proxy** - Routes subdomain requests to the corresponding static files stored in S3

![Architecture Diagram](https://i.imgur.com/r7QUXqZ.png)

## Tech Stack

| Component | Technologies |
|-----------|-------------|
| API Server | Node.js, Express, Prisma, Socket.IO |
| Database | PostgreSQL |
| Build Server | Node.js, Docker |
| Log Pipeline | Kafka, ClickHouse |
| Frontend | Next.js, Tailwind CSS, Socket.IO Client |
| Storage | AWS S3 |
| Container Orchestration | AWS ECS (optional) |

## Project Structure

```
builddesk/
├── api-server/          # Express API with Prisma ORM
│   └── prisma/          # Database schema and migrations
├── build-server/        # Dockerfile and build script
├── frontend-nextjs/     # Next.js web application
├── s3-reverse-proxy/    # Static file proxy server
└── static/              # Static assets
```

## Prerequisites

Before you begin, make sure you have the following installed:

- Node.js 18 or higher
- npm or yarn
- Docker
- PostgreSQL database
- AWS account with S3 access (for production)

Optional dependencies for the full logging pipeline:
- Kafka cluster
- ClickHouse database

## Getting Started

### Environment Variables

Create `.env` files in each service directory with the appropriate configuration.

**API Server** (`api-server/.env`):
```env
DATABASE_URL=postgresql://user:password@localhost:5432/builddesk

# AWS Configuration
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=us-east-1

# Kafka Configuration (optional)
KAFKA_BROKER=localhost:9092

# ClickHouse Configuration (optional)
CLICKHOUSE_HOST=localhost
CLICKHOUSE_USER=default
CLICKHOUSE_PASSWORD=
```

**Build Server** (injected at runtime):
```env
GIT_REPOSITORY__URL=https://github.com/user/repo.git
PROJECT_ID=project_id
DEPLOYEMENT_ID=deployment_id
```

### API Server

Navigate to the api-server directory and install dependencies:

```bash
cd api-server
npm install
```

Generate the Prisma client and start the server:

```bash
npx prisma generate
node index.js
```

The API server runs on port 9000, with a Socket.IO server on port 9002.

### Frontend

Navigate to the frontend directory and start the development server:

```bash
cd frontend-nextjs
npm install
npm run dev
```

Open your browser and go to `http://localhost:3000`.

### Build Server

Build the Docker image:

```bash
cd build-server
docker build -t builddesk-build-server .
```

To test the build server locally:

```bash
docker run --rm \
  -e GIT_REPOSITORY__URL="https://github.com/your/repo.git" \
  -e PROJECT_ID="test-project" \
  -e DEPLOYEMENT_ID="test-deployment" \
  builddesk-build-server
```

### S3 Reverse Proxy

Start the reverse proxy server:

```bash
cd s3-reverse-proxy
npm install
node index.js
```

The proxy runs on port 8000.

## Database Setup

This project uses Prisma with PostgreSQL. To set up the database:

```bash
cd api-server

# Run migrations
npx prisma migrate deploy

# Generate Prisma client
npx prisma generate
```

## Deployment

For production deployment, consider the following:

1. **Container Orchestration** - The build server is designed to run as ephemeral containers. AWS ECS is used in the original implementation, but you can adapt it to Kubernetes or any other orchestration platform.

2. **Secrets Management** - Use a secrets manager like AWS Secrets Manager or HashiCorp Vault for sensitive credentials.

3. **Database** - Use a managed PostgreSQL service for reliability.

4. **Logging** - Set up Kafka and ClickHouse for production-grade log aggregation, or substitute with your preferred logging stack.

### Running Services

| Service | Port |
|---------|------|
| API Server | 9000 |
| Socket.IO Server | 9002 |
| S3 Reverse Proxy | 8000 |
| Frontend (dev) | 3000 |

## Contributing

Contributions are welcome. To contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add some feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

Please keep changes focused and include clear descriptions in your pull requests.

## License

This project is provided as-is for educational purposes. If you plan to use this code in production, please add an appropriate license file.
