# 📘 Master Note: End-to-End ML Deployment (AWS + CI/CD)
### Project: Student Performance Indicator Stack: Python (Flask), Docker, GitHub Actions, AWS (EC2, ECR). Goal: Automate the training and deployment of an ML model from code to cloud.

🏗 Phase 1: AWS Infrastructure Setup
Before touching code, we set up the "Construction Site".

1. IAM User (The Key)
Created a specific user (e.g., cicd-deployer) with AdministratorAccess (for practice) or specific EC2/ECR permissions.

Generated: Access Key ID & Secret Access Key.

Saved in GitHub: Settings -> Secrets and variables -> Actions -> AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY.

2. ECR (The Warehouse)
Service: Elastic Container Registry.

Action: Created a private repository named student-performance.

Purpose: Stores the Docker images so EC2 can pull them later.

URI: ...dkr.ecr.us-east-1.amazonaws.com/student-performance.

3. EC2 (The Server)
OS: Ubuntu 22.04 LTS.

Instance Type:

t2.micro (Free Tier): Good for running simple apps, but crashes during heavy Docker builds.

t2.medium (Paid/Credit): Necessary for building large ML images (4GB RAM).

Security Groups (The Firewall):

Inbound Rules:

22 (SSH) - To connect terminal.

80 (HTTP) - Standard web.

443 (HTTPS) - Secure web.

8080 or 8000 (Custom TCP) - CRITICAL: Must match the port exposed in Dockerfile.

🛠 Phase 2: The CI/CD Pipeline (main.yaml)
The automation robot that moves code from GitHub to AWS.

Workflow Steps:

Integration: Checkout code -> Linting -> Unit Tests.

Delivery (Build): Log in to AWS ECR -> Build Docker Image -> Push Image to ECR.

Deployment (Run): Connect to Self-Hosted Runner -> Pull Image -> Stop old container -> Run new container.

Key Command (Run Step):

YAML

docker run -d -p 8000:8000 --ipc="host" --name=mltest \
-e 'AWS_ACCESS_KEY_ID=${{ secrets.AWS_ACCESS_KEY_ID }}' \
... <image_uri>
🏃 Phase 3: The Runner (The Bridge)
Connecting GitHub Actions to the AWS Server.

Setup:

Connect to EC2 terminal.

Download GitHub Runner package (Commands found in GitHub Repo -> Settings -> Actions -> Runners).

Config: ./config.sh --url https://github.com/... --token ...

Run:

Practice Mode: ./run.sh (Stops if terminal closes).

Production Mode: sudo ./svc.sh install -> sudo ./svc.sh start (Runs in background).

💥 Phase 4: Troubleshooting (The "Battle Scars")
These are the specific errors we fought and fixed. Memorize these.

1. Git "Divergent Branches"
Error: ! [rejected] main -> main (non-fast-forward)

Cause: GitHub had changes (like README) that local didn't have.

Fix:

Bash

git pull origin main --no-rebase  # Merge remote changes
### OR (If you don't care about remote changes)
git push -f origin main           # Force overwrite
2. "No space left on device" (Disk Full)
Error: System.IO.IOException or Docker Exit Code 125.

Cause: ML Images are huge (1.6GB+). Default EC2 disk is only 8GB.

Fix:

AWS Console: Modify Volume -> Increase to 20GB.

Terminal: Expand partition and filesystem.

Bash

sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1
Cleanup: docker system prune -a -f (Delete old junk).

3. "Container name already in use"
Error: Conflict. The container name "/mltest" is already in use.

Cause: The previous deployment failed before it could delete the old container.

Fix (Manual):

Bash

docker rm -f mltest
Fix (Automated in YAML): Add || true to the stop command so it doesn't fail if container doesn't exist.

4. "Artifacts not found" (The Missing Model)
Error: [Errno 2] No such file or directory: 'artifacts/model.pkl'

Cause: .gitignore prevented model.pkl from being pushed to GitHub, so the Docker image was empty.

Fix: Force add the ignored file.

Bash

git add -f artifacts/model.pkl
git push origin main
5. "Invalid HTTP request" (HTTPS vs HTTP)
Error: Logs show warning when accessing via Browser.

Cause: Browser forces HTTPS, but Flask/Uvicorn is running on plain HTTP.

Fix: Use http://<IP>:8000 (Explicitly HTTP) or open in Incognito mode.

🧠 Theory: Why did we do this?
Why Docker? "It works on my machine" isn't enough. Docker ensures the OS, libraries, and files are identical on AWS as they are on your laptop.

Why CICD? Manually logging into a server to type git pull is slow and risky. Automation ensures every change is tested and deployed instantly.

Why Self-Hosted Runner? AWS resources (like EC2) are private. A self-hosted runner sits inside the server, allowing GitHub to give commands to a machine it normally can't reach.

Pro Tip for Future Arpit: Next time, before you start, check your .gitignore file to make sure you aren't ignoring critical files like models (unless you are using DVC/S3 for model storage, which is a topic for next year!).