# Sober User API Development Guide

This guide explains how to work with the Sober application in different development modes, allowing you to run services both locally and in Docker containers.

## Development Modes

You can work with this application in several modes:

1. **Full Docker Mode**: All services run in Docker containers
2. **Local User API Mode**: Run the User API service locally while databases run in Docker
3. **Custom Configuration**: Connect to databases in custom locations (e.g., dev/staging environment)

## Setting Up Your Environment

### 1. Running All Services in Docker

```bash
# Start all services
docker-compose --profile full up -d

# Only start the databases (for local development)
docker-compose up -d user-mongo notification-mongo trip-mongo redis
```

### 2. Running User API Locally for Development

First, start just the databases:

```bash
docker-compose up -d user-mongo notification-mongo trip-mongo redis
```

Then, create a `.env.local` file in your sober-user-api directory:

```
PORT=3000
NODE_ENV=development
MONGODB_URI=mongodb://admin:secure_password@localhost:27017/sober-user?authSource=admin
MONGODB_USER=admin
MONGODB_PASSWORD=secure_password
JWT_SECRET=your_super_secret_key_here
NOTIFICATION_API_URL=http://localhost:3001
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_REGION=eu-central-1
AWS_S3_BUCKET_NAME=sober-user-uploads
CORS_ORIGIN=http://localhost:8080
```

Now run your NestJS application locally:

```bash
cd sober-user-api
npm install
npm run start:dev
```

### 3. Connecting to Different Environments

You can modify the `.env` file to point to different database locations:

```
# Connect to local Docker databases
USER_MONGODB_URI=mongodb://admin:secure_password@localhost:27017/sober-user?authSource=admin

# Connect to staging/production databases
USER_MONGODB_URI=mongodb://admin:secure_password@staging-db-host:27017/sober-user?authSource=admin
```

## Network Configuration

The Docker network `sober-network` is created with a fixed name to make it accessible from both containers and your local development environment. This allows your locally running services to communicate with services running in Docker.

## Docker Compose Profiles

The docker-compose.yml file uses profiles to allow selective service startup:

- `full`: Starts all services
- `user-api`: Starts just the user API service

Example usage:

```bash
# Start everything
docker-compose --profile full up -d

# Start only databases and user-api
docker-compose --profile user-api up -d

# Start only databases
docker-compose up -d
```

## Debugging

For debugging the User API service locally:

1. Start only the databases using Docker Compose
2. Run the User API with your IDE's debugging tools
3. Set the MONGODB_URI environment variable to point to localhost

Example VS Code launch configuration (.vscode/launch.json):

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug NestJS",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "start:debug"],
      "envFile": "${workspaceFolder}/.env.local",
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal"
    }
  ]
}
```

## Accessing MongoDB Data

To connect to your MongoDB instances directly:

```bash
# User MongoDB
mongo mongodb://localhost:27017/sober-user -u admin -p secure_password --authenticationDatabase admin

# Notification MongoDB
mongo mongodb://localhost:27018/sober-notification -u admin -p secure_password --authenticationDatabase admin

# Trip MongoDB
mongo mongodb://localhost:27019/sober-trip -u admin -p secure_password --authenticationDatabase admin
```