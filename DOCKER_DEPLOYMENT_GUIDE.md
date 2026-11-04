
# EV Battery Health Dashboard — Docker Deployment and Handover

**Project:** P95 — Predictive EV Battery Health & Charging Strategy  
**Client:** Geotab  
**Technology:** React, TypeScript, Flask, TensorFlow, Docker, Nginx  
**Supported platforms:** Windows and macOS  
**Recommended platform:** Windows 10/11 with Docker Desktop

This document explains how to deploy, operate, troubleshoot, and maintain the EV Battery Health Dashboard using Docker.

The application contains a React frontend, Flask prediction backend, trained LSTM models, and a cleaned sequential EV dataset.

The confirmed dataset contains:
- 11,000 electric vehicles
- 132,000 records
- 12 time steps per vehicle
- Battery health, charging, temperature, and usage information

Docker allows the application to run without installing Python, Node.js, or TensorFlow directly on the host computer.

---

## 1. What Is Included

The Docker deployment configuration includes:

- `backend/Dockerfile`: Python 3.11 container with backend dependencies and Gunicorn.
- `backend/requirements-docker.txt`: Additional Docker runtime dependencies.
- `backend/wsgi.py`: Flask WSGI entry point and health-check endpoint.
- `backend/.dockerignore`: Excludes unnecessary files from the backend build.
- `addin/Dockerfile`: Node.js build stage and Nginx runtime.
- `addin/nginx.conf.template`: Nginx configuration and API reverse proxy.
- `addin/.dockerignore`: Excludes unnecessary frontend build files.
- `docker-compose.yml`: Configuration for running frontend and backend containers.
- `DOCKER_DEPLOYMENT_GUIDE.md`: Deployment, troubleshooting, and handover instructions.

The complete application must also contain:

- `backend/app.py`
- `backend/requirements.txt`
- `backend/saved_models/`
- `backend/data/`
- `addin/package.json`
- `addin/package-lock.json`
- `addin/src/`
- Required Webpack configuration files

The Docker configuration must be used together with the approved final dashboard source code, datasets, and trained models.

---

## 2. System Architecture

The application consists of two Docker services.

### Frontend Container

- React and TypeScript dashboard
- Geotab Zenith interface
- Built using Node.js and Webpack
- Served using Nginx
- Available at `http://localhost:3001`

### Backend Container

- Python Flask application
- Gunicorn application server
- Trained LSTM models
- Battery health prediction services
- Cleaned sequential EV dataset
- Internal application port: `8080`
- Local backend endpoint: `http://localhost:5000` when exposed by Docker Compose

### Communication Architecture

```text
                 USER BROWSER
                      |
                      |
            http://localhost:3001
                      |
                      v
              REACT FRONTEND
             (Nginx Container)
                      |
                      |
                /api/ Requests
                      |
                      v
               FLASK BACKEND
            (Gunicorn Container)
                      |
                      v
               TRAINED MODELS
                      |
                      v
            SEQUENTIAL EV DATASET
                      |
                      v
              FLEET PREDICTIONS
                      |
                      v
             DASHBOARD RESULTS
```

Docker Compose manages both services through a shared Docker network.

---

## 3. Prerequisites

### 3.1 Hardware Requirements

Recommended system configuration:

- Windows 10/11 or supported macOS
- 64-bit processor
- At least 8 GB system RAM
- 16 GB RAM recommended
- At least 10 GB available storage
- Stable internet connection for initial installation

Allocate sufficient memory to Docker for TensorFlow model loading and fleet processing.

### 3.2 Required Software

- Docker Desktop
- Docker Compose (included with Docker Desktop)
- Web browser
- Windows PowerShell or macOS Terminal

Python and Node.js do not need to be installed separately.

### 3.3 Download Docker Desktop

Official download:

https://www.docker.com/products/docker-desktop/

---

# PART A — WINDOWS DEPLOYMENT

## 4. Install Docker Desktop on Windows

1. Download Docker Desktop for Windows.
2. Run the installer.
3. Enable the WSL 2 backend where supported.
4. Complete the installation.
5. Restart the computer if requested.
6. Open Docker Desktop.
7. Wait until the Docker Engine is running.

Verify the installation using Windows PowerShell:

```powershell
docker --version
```

Check Docker Compose:

```powershell
docker compose version
```

Check Docker Engine:

```powershell
docker info
```

The commands should return Docker version and server information without connection errors.

---

## 5. Prepare the Project on Windows

### Step 1 — Obtain the Final Project

Download or clone the approved final EV Battery Health Dashboard repository.

If the project is supplied as a ZIP archive, extract it.

Example project location:

```text
C:\Projects\EV-Battery-Health-Dashboard
```

### Step 2 — Verify the Directory Structure

The project should contain:

```text
EV-Battery-Health-Dashboard/
|
|-- backend/
|   |-- Dockerfile
|   |-- .dockerignore
|   |-- app.py
|   |-- wsgi.py
|   |-- requirements.txt
|   |-- requirements-docker.txt
|   |-- saved_models/
|   |-- data/
|
|-- addin/
|   |-- Dockerfile
|   |-- .dockerignore
|   |-- nginx.conf.template
|   |-- package.json
|   |-- package-lock.json
|   |-- webpack.preview.js
|   |-- src/
|
|-- docker-compose.yml
|-- DOCKER_DEPLOYMENT_GUIDE.md
```

Additional application files may be present.

### Step 3 — Open PowerShell

Open the project folder using Windows File Explorer.

Click the address bar, type:

```text
powershell
```

Press Enter.

Alternatively, navigate manually:

```powershell
cd "C:\Projects\EV-Battery-Health-Dashboard"
```

Verify the project contents:

```powershell
dir
```

Confirm that `docker-compose.yml` is present.

---

## 6. Build and Run the Application on Windows

Ensure Docker Desktop is running.

From the project root directory, execute:

```powershell
docker compose up --build
```

Docker Compose will:

1. Read the Compose configuration.
2. Build the backend Docker image.
3. Install Python and machine-learning dependencies.
4. Copy backend application files and required model assets.
5. Build the React frontend.
6. Install frontend dependencies.
7. Compile the frontend using Webpack.
8. Configure Nginx.
9. Create and start both containers.

The initial build may take several minutes or longer.

Keep the PowerShell window open while the application is running.

### Verify Container Status

Open another PowerShell window in the project directory.

Run:

```powershell
docker compose ps
```

Both frontend and backend services should be running.

The services can also be viewed in:

**Docker Desktop → Containers**

---

## 7. Verify Backend Model Loading

During startup, the backend loads the trained machine-learning models and processes the cleaned sequential EV dataset.

Open a separate PowerShell window in the project directory.

Run:

```powershell
docker compose logs --tail=50 backend
```

Successful processing may display messages similar to:

```text
Loading models and data
(Cleaned Sequential EV Dataset)...

Dataset loaded: 132,000 rows
11,000 vehicles | 12 timesteps

Ready — 11,000 vehicles scored.
```

These messages confirm that the dataset was loaded and fleet predictions were generated.

The initial processing time depends on available CPU, memory, and model-loading performance.

---

## 8. Access the Dashboard

Open a browser and navigate to:

**http://localhost:3001**

The dashboard contains the following sections:

### Fleet Overview

Displays:

- Average battery State of Health
- Vehicles below the configured SoH threshold
- High-stress vehicles
- Vehicles requiring attention
- Model prediction error
- Battery health distribution
- Battery stress visualisations

### Fleet Management

Provides:

- Vehicle fleet records
- Configurable SoH threshold
- Vehicle health status
- Estimated battery replacement information
- Estimated service information
- Vehicle filtering and selection

### Vehicle Dashboard

Displays detailed battery health information for an individual vehicle.

### What-if Analysis

Allows users to explore how changes in relevant battery and usage variables affect predicted battery health.

### Initial Loading

The dashboard may display:

```text
Loading models and data...
Loading fleet predictions...
```

This occurs while the backend initialises the models and prepares the prediction results.

If the backend is still processing, allow it to complete before refreshing the page.

If loading continues unexpectedly, inspect the backend logs.

---

## 9. Verify Backend Health

When the backend port is exposed on `localhost:5000`, open:

**http://localhost:5000/health**

A healthy backend should return a response similar to:

```json
{"status":"ok"}
```

This health check verifies that the Flask service is responding.

It does not independently confirm that all machine-learning predictions have completed.

To verify prediction readiness, inspect the backend logs and test the dashboard.

---

## 10. Stop the Application on Windows

In the PowerShell window running Docker Compose, press:

**Ctrl + C**

Then run:

```powershell
docker compose down
```

This stops and removes the application containers.

The Docker images and project source files remain available.

---

## 11. Restart the Application on Windows

Open Docker Desktop and wait for the engine to start.

Open PowerShell in the project directory.

Run:

```powershell
docker compose up
```

The application will restart using the existing Docker images.

To rebuild after source-code or configuration changes:

```powershell
docker compose up --build
```

To run the containers in the background:

```powershell
docker compose up -d
```

To stop background containers:

```powershell
docker compose down
```

---

# PART B — macOS DEPLOYMENT

## 12. Install Docker Desktop on macOS

1. Open the official Docker Desktop website.
2. Download the installer for the Mac processor.
3. Select Apple Silicon or Intel as appropriate.
4. Open the downloaded `.dmg` file.
5. Move Docker into Applications.
6. Launch Docker Desktop.
7. Complete the initial configuration.
8. Wait until Docker Engine is running.

Open Terminal.

Verify Docker:

```bash
docker --version
```

Verify Docker Compose:

```bash
docker compose version
```

Verify Docker Engine:

```bash
docker info
```

---

## 13. Prepare the Project on macOS

Download or clone the approved final dashboard project.

Extract the archive if necessary.

Open Terminal and navigate to the project directory.

Example:

```bash
cd ~/Desktop/EV-Battery-Health-Dashboard
```

Verify the contents:

```bash
ls
```

Confirm that the following exist:

- `backend/`
- `addin/`
- `docker-compose.yml`

Ensure that all required model files and datasets are included.

---

## 14. Build and Run on macOS

Ensure Docker Desktop is running.

Execute:

```bash
docker compose up --build
```

Docker will build and start both application containers.

The initial build may take longer on older hardware.

Verify the services:

```bash
docker compose ps
```

Check backend logs:

```bash
docker compose logs --tail=50 backend
```

Wait for the dataset and models to finish loading.

---

## 15. Access the Dashboard on macOS

Open Safari, Chrome, or another supported browser.

Navigate to:

**http://localhost:3001**

Verify that:

1. Fleet Overview loads.
2. Fleet Management displays vehicle information.
3. SoH filtering works.
4. Individual vehicle details are accessible.
5. What-if Analysis returns results.

When the backend port is exposed, the health endpoint can also be checked at:

**http://localhost:5000/health**

---

## 16. Stop and Restart on macOS

Stop the foreground application using:

**Control + C**

Then run:

```bash
docker compose down
```

Restart:

```bash
docker compose up
```

Rebuild:

```bash
docker compose up --build
```

Run in the background:

```bash
docker compose up -d
```

### macOS Compatibility

Apple Silicon and Intel processors may require different container image architectures.

If the selected TensorFlow or Python dependencies do not support the processor architecture, compatible dependencies or Docker platform settings may be required.

The macOS deployment should be validated on the intended target hardware before production use.

---

# PART C — DOCKER CONFIGURATION

## 17. Backend Dockerfile

**File:** `backend/Dockerfile`

The backend Dockerfile is responsible for:

- Using a compatible Python runtime
- Setting environment variables
- Creating the application working directory
- Installing Python dependencies
- Copying the Flask application
- Including required model and dataset files
- Configuring the backend service
- Running the application using Gunicorn

The backend uses port `8080` inside the container.

The Docker Compose configuration can expose this service locally through port `5000`.

---

## 18. Frontend Dockerfile

**File:** `addin/Dockerfile`

The frontend Dockerfile uses a multi-stage build.

### Build Stage

Uses Node.js to:

- Install frontend dependencies
- Copy application source code
- Run Webpack
- Generate production frontend assets

### Runtime Stage

Uses Nginx to:

- Serve the compiled React application
- Handle frontend routing
- Forward configured API requests to Flask

The frontend is accessed through port `3001` on the host.

---

## 19. Docker Compose Configuration

**File:** `docker-compose.yml`

Docker Compose coordinates the frontend and backend containers.

Its responsibilities include:

- Building the images
- Starting the containers
- Configuring container networking
- Mapping host and container ports
- Supplying environment variables
- Managing service lifecycle

The two services communicate through Docker's internal network.

---

## 20. Nginx Configuration

**File:** `addin/nginx.conf.template`

Nginx serves the compiled React application.

It also forwards requests under the configured `/api/` path to the Flask backend.

This allows the frontend to communicate with the backend through the same application entry point.

---

## 21. Docker Ignore Configuration

**Files:**

- `backend/.dockerignore`
- `addin/.dockerignore`

These files exclude unnecessary files from the Docker build.

The frontend configuration should exclude local `node_modules` to prevent Windows or macOS dependencies from overwriting Linux dependencies installed inside Docker.

Example frontend exclusions:

```text
node_modules/
dist/
.git/
.env
.env.*
.DS_Store
*.log
```

Backend exclusions should avoid removing required trained models or datasets.

---

# PART D — TROUBLESHOOTING

## 22. Docker Engine Is Not Running

### Error

```text
failed to connect to the docker API
```

### Cause

Docker Desktop is not running or the Docker Engine has not started.

### Resolution

1. Open Docker Desktop.
2. Wait until the engine is running.
3. Verify:

```bash
docker info
```

4. Retry:

```bash
docker compose up --build
```

---

## 23. Docker Hub Connection Error

### Error

```text
lookup registry-1.docker.io:
no such host
```

### Cause

Docker cannot access Docker Hub due to DNS, network, VPN, proxy, or firewall issues.

### Resolution

1. Check the internet connection.
2. Restart Docker Desktop.
3. Verify DNS and proxy settings.
4. Test downloading the backend base image:

```bash
docker pull python:3.11-slim
```

5. Retry:

```bash
docker compose up --build
```

---

## 24. Frontend Webpack Build Error

### Error

```text
webpack.js: not found
```

### Possible Cause

The frontend build may be using incompatible or incomplete dependencies.

Local `node_modules` files may also interfere with Docker's Linux dependency installation.

### Resolution

Confirm that `addin/.dockerignore` contains:

```text
node_modules/
```

Rebuild the frontend:

```bash
docker compose build --no-cache frontend
```

Start the containers:

```bash
docker compose up
```

If the error continues, check the frontend dependency definitions and Docker build logs.

---

## 25. Backend Returns a 504 Gateway Timeout

### Error

```text
Could not reach the prediction backend
Request failed (504)
```

### Possible Causes

- Backend models are still loading.
- Fleet predictions are still being processed.
- Backend is not responding.
- Backend container has stopped.
- Nginx request timeout has been exceeded.

### Resolution

Check the running services:

```bash
docker compose ps
```

Inspect backend logs:

```bash
docker compose logs --tail=100 backend
```

If processing is still active, wait for completion.

If the backend has stopped or is reporting errors, investigate the logs before restarting.

After the backend becomes ready, refresh the dashboard.

---

## 26. Dashboard Remains on the Loading Screen

### Symptom

```text
Loading fleet predictions...
```

### Explanation

The backend loads trained machine-learning models and prepares fleet prediction results.

This may require additional startup time.

### Resolution

Inspect the backend logs:

```bash
docker compose logs --tail=100 backend
```

Look for:

```text
Dataset loaded
Ready — 11,000 vehicles scored.
```

If the backend reports readiness but the frontend remains stuck, check the API requests and Nginx configuration.

---

## 27. Insufficient Memory

### Symptoms

- Backend container exits unexpectedly.
- Model loading fails.
- Application becomes unresponsive.
- Docker reports memory-related errors.

### Resolution

1. Check Docker Desktop resource usage.
2. Close unnecessary applications.
3. Increase memory available to Docker where supported.
4. Restart the application:

```bash
docker compose down
docker compose up
```

For Windows using WSL 2, memory allocation may be controlled through WSL configuration.

---

## 28. Port 3001 Is Already in Use

### Error

```text
port is already allocated
```

### Windows

Identify the process using the port:

```powershell
netstat -ano | findstr :3001
```

### macOS

```bash
lsof -i :3001
```

Stop the conflicting application or modify the frontend port mapping in `docker-compose.yml`.

If the port changes, use the updated address to access the dashboard.

---

## 29. TensorFlow Retracing Warnings

### Warning

```text
TensorFlow function retracing
```

### Explanation

TensorFlow may generate warnings when prediction functions are repeatedly traced.

These warnings do not necessarily indicate application failure.

However, excessive retracing can affect prediction performance.

### Resolution

Verify whether predictions complete successfully.

If performance problems persist, review model inference logic, input shapes, and repeated prediction function creation.

---

# PART E — GOOGLE CLOUD RUN DEPLOYMENT

## 30. Cloud Deployment Overview

The Docker configuration provides a foundation for deploying the application to Google Cloud Run.

Cloud Run can host containerised applications without requiring the client to manage individual virtual machines.

The local Docker Compose configuration is intended for local deployment and testing.

Cloud Run requires additional configuration and independent deployment verification.

---

## 31. Google Cloud Prerequisites

The organisation responsible for cloud hosting requires:

- Google Cloud account
- Google Cloud project
- Enabled billing
- Google Cloud CLI
- Cloud Run API
- Cloud Build API
- Artifact Registry API
- Appropriate IAM permissions

Official documentation:

https://cloud.google.com/run/docs

---

## 32. Prepare the Backend for Cloud Run

The backend must be configured to:

1. Listen on the port provided through the `PORT` environment variable.
2. Bind to `0.0.0.0`.
3. Include all required dependencies.
4. Access the trained models and datasets.
5. Use suitable CPU and memory settings.
6. Support the required startup and request timeouts.
7. Expose appropriate application health endpoints.

Because the application processes a large sequential EV dataset, startup time and resource requirements must be evaluated before production deployment.

---

## 33. Build and Publish Container Images

The deployment process involves:

1. Authenticate using Google Cloud CLI.
2. Select the target Google Cloud project.
3. Enable the required Google Cloud services.
4. Create or select an Artifact Registry repository.
5. Build the backend container image.
6. Publish the image to Artifact Registry.
7. Deploy the image to Cloud Run.
8. Configure service environment variables and permissions.
9. Verify application startup and health.

The exact commands depend on the client's Google Cloud project, region, repository, and security requirements.

---

## 34. Deploy the Frontend

The frontend may be deployed as a separate Cloud Run service.

The deployment must account for:

- React production build
- Nginx configuration
- Backend API address
- HTTPS routing
- CORS configuration
- Authentication and access control

The frontend must be configured to communicate with the deployed backend rather than relying on local Docker service names.

---

## 35. Verify Cloud Deployment

After deployment, verify:

- Backend service starts successfully.
- Trained models load.
- Dataset is available.
- Prediction endpoints respond correctly.
- Frontend connects to the backend.
- Fleet Overview displays results.
- Fleet Management functions correctly.
- Vehicle details are accessible.
- What-if Analysis returns predictions.
- Resource allocation is sufficient.
- Access permissions meet client requirements.

**Deployment status:** Local Docker deployment has been tested on Windows. Cloud Run deployment must be configured and verified separately before being considered complete.

---

# PART F — MAINTENANCE AND HANDOVER

## 36. Common Docker Commands

### Build and Start

```bash
docker compose up --build
```

### Start Existing Containers

```bash
docker compose up
```

### Start in Background

```bash
docker compose up -d
```

### Stop and Remove Containers

```bash
docker compose down
```

### View Running Services

```bash
docker compose ps
```

### View Backend Logs

```bash
docker compose logs backend
```

### View Frontend Logs

```bash
docker compose logs frontend
```

### View Recent Backend Logs

```bash
docker compose logs --tail=50 backend
```

### Follow Live Logs

```bash
docker compose logs -f
```

### Rebuild Frontend Without Cache

```bash
docker compose build --no-cache frontend
```

### Rebuild Backend Without Cache

```bash
docker compose build --no-cache backend
```

---

## 37. Updating the Application

When application code, dependencies, or configuration files change:

1. Update the relevant project files.
2. Verify that required model and dataset files remain available.
3. Rebuild the containers:

```bash
docker compose up --build
```

4. Inspect the backend logs.
5. Verify dashboard functionality.

If the trained models or datasets are packaged directly into the backend image, rebuild that image after updating them.

---

## 38. Security Considerations

Before deploying to a production environment:

- Avoid committing passwords or API credentials.
- Use secure environment-variable or secret-management mechanisms.
- Restrict backend access where appropriate.
- Configure HTTPS for externally accessible services.
- Review Cloud Run authentication settings.
- Apply least-privilege IAM permissions.
- Keep Docker images and dependencies updated.
- Protect datasets and model files according to client requirements.

The local Docker deployment should not be assumed to provide production-ready security controls without further configuration.

---

## 39. Final Handover Package

The final handover should include:

- Complete application source code
- GitHub repository
- React frontend
- Flask backend
- Trained LSTM models
- Required datasets
- Frontend Dockerfile
- Backend Dockerfile
- Docker Compose configuration
- Nginx configuration
- Dependency files
- README documentation
- Docker deployment guide
- Model and dataset setup information

The repository and deployment package should contain all files necessary to reproduce the approved application, subject to any separately documented dataset or credential access requirements.

---

## 40. Deployment Verification Checklist

### Windows

- [ ] Docker Desktop installed
- [ ] Docker Engine running
- [ ] Project files available
- [ ] Docker images build successfully
- [ ] Backend container starts
- [ ] Frontend container starts
- [ ] Dataset loads successfully
- [ ] Models generate predictions
- [ ] Dashboard opens at localhost:3001
- [ ] Fleet Overview works
- [ ] Fleet Management works
- [ ] Vehicle Dashboard works
- [ ] What-if Analysis works

### macOS

- [ ] Compatible Docker Desktop installed
- [ ] Docker Engine running
- [ ] Required container architectures supported
- [ ] Images build successfully
- [ ] Both services start
- [ ] Backend loads models and dataset
- [ ] Dashboard functions correctly

### Cloud Deployment

- [ ] Google Cloud project configured
- [ ] Required APIs enabled
- [ ] Container images published
- [ ] Cloud Run services deployed
- [ ] Backend health verified
- [ ] Frontend/backend communication verified
- [ ] Security settings reviewed
- [ ] Resource allocation validated
- [ ] Application functionality tested

---

## 41. Conclusion

The EV Battery Health Dashboard has been prepared for containerised deployment using Docker and Docker Compose.

The application consists of a React frontend served through Nginx and a Flask backend responsible for battery health predictions.

The backend uses trained machine-learning models and a cleaned sequential EV dataset containing 11,000 vehicles and 132,000 records.

The Windows Docker deployment has been tested successfully, including container startup, backend fleet processing, and dashboard visualisation.

This guide provides the installation, deployment, verification, troubleshooting, and maintenance information required to reproduce the local application environment.

The Docker configuration also provides a foundation for future cloud hosting, subject to additional cloud configuration, compatibility testing, and production validation.

---

**End of Docker Deployment and Handover Guide**
