# PromptLib
PromptLib is a lightweight and flexible tool for managing your prompt library. It helps developers, researchers, and AI enthusiasts organize, store, and reuse prompts with ease. Designed for productivity and collaboration, PromptLib makes working with prompts more structured and efficient

## 🚀 PromptLib Deployment Guide

This repository contains the configuration files required to deploy the PromptLib stack (Frontend, Backend, and Database) using Docker.
Note: This repository does not contain the source code. It provides the pre-built Docker images and orchestration logic for easy deployment.

### 📋 Prerequisites

Before you begin, ensure you have the following installed on your system:
- Docker (20.10.x or higher)
- Docker Compose (v2.x or higher)

### 🛠️ Getting Started

#### Clone this repository

```Bash
https://github.com/thuanvu301103/PromptLib.git
cd PromptLib
```

#### Configure Environment Variables

Copy the example environment file and adjust the values if necessary.
```Bash
cp .env.example .env
```

Open the .env file and verify the settings:
- `MONGODB_LOCAL_PORT`: The port on your host machine to access MongoDB (default: 27018).
- `PORT`: The port to access the Frontend UI (default: 3001).
- `NEXT_PUBLIC_API_URL`: The URL where the Frontend can reach the Backend API (default: http://localhost:3000).

#### Deploy with Docker Compose

Run the following command to pull the images from GitHub Container Registry (GHCR) and start the services in detached mode:

```Bash
docker-compose up -d
```

### 🏗️ Architecture Overview

The system consists of three main services:

| Service | Image Source | Port (Host) | Description | 
| --- | --- | --- | --- |
| UI | ghcr.io/thuanvu301103/promptlib-frontend | 3001 | Web Interface |
| API | ghcr.io/thuanvu301103/promptlib-api | 3000 | Backend Service | 
| Database | mongo:latest | 27018 | MongoDB for data persistence | 

### 🛑 Troubleshooting

Common Issues:
- Port Conflict: If ports 3000, 3001, or 27018 are already in use, change the mapping in your .env file.
- Connectivity: Ensure the NEXT_PUBLIC_API_URL is accessible from your browser (client-side), as Next.js calls the API from the user's browser.
- Database Auth: If you change MONGODB_USER or MONGODB_PASSWORD after the first run, you may need to clear the volume: docker-compose down -v.

### 🔒 Security & Privacy

The source code for this project is hosted in a Private Repository.The Docker images are built using Multi-stage builds to ensure that no source code or sensitive build-time artifacts are included in the final distribution.No sensitive credentials (API Keys, Production Secrets) are hardcoded into the images.

### 📧 Support
If you encounter any issues during deployment, please open an Issue in this repository or contact the maintainer at vungocthuan1234@gmail.com.

---

## 🚀 PromptLib API

PromptLib API is a specialized backend service designed to streamline the management of LLM prompt templates. It provides a centralized repository for creating, versioning, and retrieving prompts, allowing developers to decouple prompt engineering from core application logic.

### 🏗️ System Architecture

The repository uses a containerized multi-service architecture:

- API Service (`promptlib_api`): The core engine that handles prompt logic, variable injection, and template distribution. It connects to MongoDB using the credentials defined in the environment.
- Database Service (`promptlib_mongodb`): A MongoDB instance that provides persistent storage for all your prompt templates.

### 🛠️ Tech Stack

- Database: MongoDB
- Orchestration: Docker Compose
- Network: Private bridge network for secure inter-service communication.

### 📥 Setup and Execution Guide

#### Prepare the Environment File

Create a file named `.env` in your root directory and paste the following configuration:

```Bash
# MongoDB Configuration
MONGODB_USER=admin
MONGODB_PASSWORD=password123
MONGODB_DATABASE=promptlib
MONGODB_LOCAL_PORT=27018

# API Configuration
DATABASE_HOST=mongodb
FRONTEND_URL=http://localhost:3001
```

#### Docker Compose Configuration

Ensure your `docker-compose.yml` is set up to consume these variables:

```YAML
version: '3.8'

services:
  mongodb:
    image: mongo:latest
    container_name: promptlib_mongodb
    restart: always
    ports:
      - "${MONGODB_LOCAL_PORT:-27018}:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGODB_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGODB_PASSWORD}
      MONGO_INITDB_DATABASE: ${MONGODB_DATABASE}
    volumes:
      - mongodb_data:/data/db
    networks:
      - prompt_network

  api:
    image: thuanvu301103/promptlib-api:latest
    container_name: promptlib_api
    restart: always
    ports:
      - "3000:3000"
    environment:
      DATABASE_HOST: ${DATABASE_HOST}
      MONGODB_USER: ${MONGODB_USER}
      MONGODB_PASSWORD: ${MONGODB_PASSWORD}
      MONGODB_DATABASE: ${MONGODB_DATABASE}
      FRONTEND_URL: ${FRONTEND_URL}
    depends_on:
      - mongodb
    networks:
      - prompt_network

networks:
  prompt_network:
    driver: bridge

volumes:
  mongodb_data:
```

### Running the Application

To build and start the entire stack:

```Bash
# Build and start in detached mode
docker-compose up --build -d
```

### Verification

- API: Access the service at `http://localhost:3000`.
- Swagger: Access the service at `http://localhost:3000/api`.
- Logs: Run `docker logs -f promptlib_api` to see real-time interaction between the API and MongoDB.

### 💡 Important Notes

- Data Persistence: Your prompt data is stored in the `mongodb_data` volume. It will survive a `docker-compose down`, but will be deleted if you use `docker-compose down -v`.
- CORS: The `FRONTEND_URL` environment variable is used to configure CORS in the FastAPI app, allowing your frontend (on port 3001) to communicate with this API.
- Security: Change the `MONGODB_PASSWORD` for any environment beyond local development.

## 🚀 PromptLib UI

`promptlib-ui` is the official web-based user interface for the PromptLib API. It provides an intuitive, user-friendly dashboard for prompt engineers and developers to manage, test, and version their LLM prompt templates without interacting directly with the backend code or database.

### ✨ Key Features

- Prompt Templates Management Dashboard: A centralized UI to create, browse, and organize all your stored prompt templates.
- Version History Browser: Easily navigate through different versions of a prompt and compare changes visually.

### 🏗️ Execution Guide

#### Pulling the image

```Bash
docker pull thuanvu301103/promptlib-ui:latest
```

#### Running with Docker Compose

To run the UI alongside your API, add this service to your docker-compose.yml:

```YAML  
ui:
    image: thuanvu301103/promptlib-ui:latest
    container_name: promptlib_ui
    restart: always
    ports:
      - "3001:3000"
    environment:
      # URL of your running PromptLib API
      - NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL:-http://localhost:3000}
    networks:
      - prompt_network
```

### 📂 Configuration

The UI is highly configurable through environment variables:

| Variable | Description | Default | 
| --- | --- | --- |
| NEXT_PUBLIC_API_URL | The endpoint where the UI can reach your PromptLib API | http://localhost:3000 | 
| NODE_ENV | Environment mode (production/development) | production |
