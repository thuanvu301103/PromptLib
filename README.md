# PromptLib
PromptLib is a lightweight and flexible tool for managing your prompt library. It helps developers, researchers, and AI enthusiasts organize, store, and reuse prompts with ease. Designed for productivity and collaboration, PromptLib makes working with prompts more structured and efficient

## 🚀 PromptLib API: Repository Overview

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
