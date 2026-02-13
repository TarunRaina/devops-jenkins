# 🚀 Jenkins + Docker CI/CD Pipeline (Hands-on Implementation)

Author: Tarun Raina  

---

## 📌 Overview

This project demonstrates a complete DevOps workflow:

- Linux shell scripting
- Docker verification and inspection
- React application setup and testing
- Git version control
- Jenkins-based CI/CD pipeline
- Docker image build and automated container deployment

The final result is a **3-stage Jenkins pipeline**:

1. Clone source code
2. Build Docker image
3. Run Docker container

Triggered manually at Stage 1, then fully automated downstream.

---

## 🧱 Environment

- Ubuntu VM
- Docker 29.2.1
- Node.js v18
- OpenJDK 21
- Jenkins
- GitHub

---

## Phase 0 — Initial Script & Environment Validation

### Creating and Running a Shell Script
```
./sample-script.sh
```
Permission denied → fixed via:

```
chmod +x sample-script.sh
./sample-script.sh
```
The script outputs:

Docker version

Docker images

Docker containers

Java version

This validated:

Docker daemon running

Images present

Containers listed

Java installed (required for Jenkins)

Example output:
```
Docker version 29.2.1
docker images
docker ps -a
openjdk 21.0.10
```
Purpose: sanity check before moving into CI/CD.

Phase 1 — React Application Setup
Clone React Repository
```
git clone https://github.com/TarunRaina/react-site-sample-devops.git
cd react-site-sample-devops
```

Install Dependencies
```
npm i
```

Warnings appeared due to Node version mismatch (React Router expects Node ≥20), but the install succeeded.

Run Development Server
```
npm run dev
For LAN exposure:

npx vite --host
```
App became accessible at:

http://<VM-IP>:5173
![description](screenshots/website.png)


Modify Package Script + Push Changes
```
git add package.json
git commit -m "Update dev script to host on all interfaces for Linux"
git push origin main
```
This validated:

Local dev flow

Git commit lifecycle

Remote push

Phase 2 — Jenkins + Docker Preparation
Jenkins Docker Permission Fix
Allow Jenkins to execute Docker:

```
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo systemctl restart docker
```

Verify:
```
sudo su - jenkins
docker ps
```
No permission errors → Jenkins is Docker-capable.

![description](screenshots/test-jenkins-run.png)

Phase 3 — Application for CI/CD
A minimal Node + Express app was prepared with:

index.html

server.js

package.json

Dockerfile

Pushed to GitHub.

This becomes the pipeline input.

Phase 4 — Jenkins Pipeline Architecture
Instead of a Jenkinsfile, three freestyle jobs were created:

🔹 Job 1 — clone-job (Manual Trigger)
Purpose: pull source code.

SCM → Git

Branch: */main

Build step:
```
echo Clone successful
ls
```

Workspace:
```
/var/lib/jenkins/workspace/clone-job
```
![description](screenshots/clone-job-item1.png)


🔹 Job 2 — docker-image-job (Triggered by clone-job)
Trigger:

☑ Build after other projects are built
Upstream: clone-job

Build step:
```
echo "Starting Docker image build..."

cd /var/lib/jenkins/workspace/clone-job

echo "Building Docker image from cloned repo..."

docker build -t pipeline-app .

echo "Docker image pipeline-app created successfully."
This produces:

pipeline-app:latest

```


![description](screenshots/build-img-item1.png)
![description](screenshots/trig-item3.png)


🔹 Job 3 — docker-run-job (Triggered by docker-image-job)
Trigger:

☑ Build after other projects are built
Upstream: docker-image-job

Build step:

```
echo "Stopping old container if exists..."

docker rm -f pipeline-app || true

echo "Starting new container..."

docker run -d -p 80:3000 --name pipeline-app pipeline-app

echo "Application deployed successfully."
```
This:

Removes old container

Runs new one

![description](screenshots/run-job-item3.png)


Phase 5 — Execution Flow
Only Job 1 is started manually:

clone-job
Then automatically:

clone-job
   ↓
docker-image-job
   ↓
docker-run-job
One click.

Three stages.

Zero micromanagement.

Phase 6 — Validation
Jenkins Logs
Clone successful

Docker image built

Container deployed

Docker Verification
docker ps
Output:
```
CONTAINER ID   IMAGE          PORTS
pipeline-app  pipeline-app  0.0.0.0:80->3000
```
Container alive.

Browser Test
http://<VM-IP>
Application loads successfully.

![description](screenshots/container-created.png)

![description](screenshots/running.png)


✅ Result
A complete CI/CD pipeline:

GitHub → Jenkins

Jenkins → Docker build

Docker → Live container

Manual entry at Stage 1, automated delivery thereafter.

This implements:

Continuous Integration

Automated Image Build

Continuous Deployment
