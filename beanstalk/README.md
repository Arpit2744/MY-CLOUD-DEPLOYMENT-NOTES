# Master Note: AWS Elastic Beanstalk Deployment (CD Pipeline)
Project: Student Performance Indicator Stack: Python (Flask), AWS Elastic Beanstalk, AWS CodePipeline. Goal: Deploy the application using a Managed Service (PaaS) to avoid managing servers manually.

⚙️ Phase 1: Code Configuration (The "Magic" Setup)
Beanstalk needs specific files to understand how to run Python code.

1. The Entry Point (application.py)
Rule: Elastic Beanstalk looks for a file named application.py and a callable object named application by default.

Action:

Renamed app.py ➔ application.py.

Inside code: Changed app = Flask(__name__) ➔ application = Flask(__name__).

Why? If you keep it as app.py, WSGI (the web server) won't find it and gives a 502 Bad Gateway.

2. The Config Folder (.ebextensions)
Folder Name: .ebextensions (Must be in root).

File Name: python.config

Content:

YAML

option_settings:
  "aws:elasticbeanstalk:container:python":
    WSGIPath: application:application
Purpose: Tells AWS, "Look inside the file application.py and find the function application to start the server."

🏗 Phase 2: AWS Infrastructure (Beanstalk)
Setting up the environment.

1. Create Application
Service: Elastic Beanstalk.

Platform: Python 3.x (Linux).

Application Code: Started with "Sample Application" (to verify platform works) or uploaded code directly.

2. Configure Environment
Presets: Selected "Single Instance" (Free Tier eligible).

Capacity: Auto-scaling triggers (optional, usually kept default for practice).

Health: Checks if the app responds with "200 OK".

🚀 Phase 3: The Pipeline (AWS CodePipeline)
Connecting GitHub to Beanstalk for automatic updates.

Workflow:

Source:

Provider: GitHub (Version 1 or 2).

Repo: ML-Project.

Branch: main.

Trigger: "Start pipeline on source code change" (Auto-deploy).

Build: (Skipped/Optional)

Note: Simple Python apps often skip the "Build" phase because Python is interpreted, not compiled.

Deploy:

Provider: AWS Elastic Beanstalk.

Application: StudentPerformance.

Environment: StudentPerformance-env.

💥 Phase 4: Troubleshooting (Common Errors)
1. "502 Bad Gateway" (Nginx Error)
Cause: The server started, but couldn't find your Python app.

Fix: Checked application.py renaming. Ensured application = Flask(__name__) exists. Verified .ebextensions/python.config formatting.

2. "Degraded" Health Status
Cause: The application is taking too long to respond or crashing silently.

Fix: Checked Logs (Request Logs -> Last 100 Lines). Usually a missing library in requirements.txt.

3. "ModuleNotFoundError"
Cause: requirements.txt was missing or had the wrong package names.

Fix: Ran pip freeze > requirements.txt locally and pushed to GitHub. Beanstalk installs these automatically.