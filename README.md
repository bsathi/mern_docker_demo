Markdown
# Library MERN Stack Application with Docker & Nginx

A full-stack MERN (MongoDB, Express, React, Node.js) application orchestrated with Docker and Nginx reverse proxy. This repository contains the complete Dockerized setup for local development, supporting both manual container management and single-command orchestration via Docker Compose.

---

## 🛠️ Architecture Overview

The application consists of 5 microservices running on a shared Docker bridge network (`library-mern-api`):

1. **Nginx (`mern_library_nginx_nginx_1`)**: Reverse proxy listening on port `8080`. Routes web traffic to the React client and API calls (`/api`) to the Express backend.
2. **React Client (`library_mern_frontend`)**: Frontend UI running on Node 16 Alpine (`http://localhost:3000`).
3. **Express API (`library_mern_nginx`)**: Node.js backend REST service running on port `5000`.
4. **MongoDB (`mern_library_nginx_mongodb_1`)**: NoSQL database running on port `27017`.
5. **Mongo Express (`mern_library_nginx_mongo-express_1`)**: Web-based MongoDB administration dashboard running on port `8081`.

---

## 🚀 Key Fixes & Modifications

* **Node 16 Alpine Downgrade**: Downgraded the frontend client base image from Node 20 to Node 16 (`node:16-alpine`) to prevent `ERR_OSSL_EVP_UNSUPPORTED` errors caused by OpenSSL 3.0 incompatibility with legacy `react-scripts` v3.
* **Volume Cache Permissions**: Configured `-e DISABLE_ESLINT_PLUGIN=true` and `-e CHOKIDAR_USEPOLLING=true` on the frontend client to resolve Linux WSL2 file permission locks (`EACCES: permission denied`) on `.cache` and `.eslintcache`.
* **Strict Container Naming**: Standardized container naming conventions (`library_mern_frontend`) to ensure seamless upstream DNS resolution within `nginx/default.conf`.

---

## 🏎️ Quick Start: Docker Compose

### 1. Build and Start the Stack
```bash
docker compose up --build -d
2. Verify Running Services
Bash
docker compose ps
3. Access Applications
Library Web Application: http://localhost:8080

Mongo Express Dashboard: http://localhost:8081

4. Stop the Application
Bash
docker compose down
🛠️ Manual Step-by-Step Container Setup
If you prefer to build and execute each container individually using docker run:

Step 1: Create Shared Docker Network
Bash
docker network create library-mern-api
Step 2: Launch MongoDB
Bash
docker run -d \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  --net library-mern-api \
  --name mern_library_nginx_mongodb_1 \
  mongo
Step 3: Launch Mongo Express
Bash
docker run -d \
  -p 8081:8081 \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
  -e ME_CONFIG_MONGODB_SERVER=mern_library_nginx_mongodb_1 \
  -e ME_CONFIG_BASICAUTH=false \
  --net library-mern-api \
  --name mern_library_nginx_mongo-express_1 \
  mongo-express
Step 4: Build and Launch Express Backend API
Bash
cd ~/docker/mern_docker_demo/server
docker build -t mern_library_nginx_library-api .

docker run -d \
  -p 5000:5000 \
  -e MONGO_URI=mongodb://admin:password@mern_library_nginx_mongodb_1:27017 \
  --net library-mern-api \
  --name library_mern_nginx \
  mern_library_nginx_library-api
Step 5: Build and Launch React Frontend Client
Bash
cd ~/docker/mern_docker_demo/client
docker build -t mern_library_nginx_client .

docker run -d \
  -p 3000:3000 \
  -v "$(pwd)":/app \
  -v /app/node_modules \
  -e DISABLE_ESLINT_PLUGIN=true \
  -e CHOKIDAR_USEPOLLING=true \
  --net library-mern-api \
  --name library_mern_frontend \
  mern_library_nginx_client
Step 6: Build and Launch Nginx Reverse Proxy
Bash
cd ~/docker/mern_docker_demo/nginx
docker build -t mern_library_nginx_nginx .

docker run -d \
  -p 8080:80 \
  --net library-mern-api \
  --name mern_library_nginx_nginx_1 \
  mern_library_nginx_nginx
📊 Viewing Logs and Troubleshooting
Viewing Logs
To monitor logs across individual services or the entire stack:

Bash
# View aggregated Compose logs (all services)
docker compose logs -f

# View specific service logs individually
docker logs -f mern_library_nginx_nginx_1    # Nginx Proxy
docker logs -f library_mern_frontend          # React Client
docker logs -f library_mern_nginx             # Express API
docker logs -f mern_library_nginx_mongodb_1   # MongoDB
docker logs -f mern_library_nginx_mongo-express_1 # Mongo Express
Common Issues & Solutions
ERR_OSSL_EVP_UNSUPPORTED: Ensure client/Dockerfile uses FROM node:16-alpine. Node 17+ uses OpenSSL 3.0, which breaks legacy Webpack/react-scripts v3.

EACCES: permission denied for .cache: Caused by root-owned volume mounts in WSL2. Fixed by setting -e DISABLE_ESLINT_PLUGIN=true and -e CHOKIDAR_USEPOLLING=true.

Nginx [emerg] host not found: Ensure the frontend container name is strictly set to library_mern_frontend so Nginx DNS resolution matches /etc/nginx/conf.d/default.conf.

HTTP 404 on API Requests: Always access the application via http://localhost:8080 (Nginx proxy) rather than port 3000 directly so requests route through to the API on port 5000.
