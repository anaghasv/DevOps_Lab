# Continuous Integration (CI) Automation with Jenkins

## Overview

This project introduces Continuous Integration (CI) and demonstrates how to install and run Jenkins using Docker.

Jenkins is deployed as a Docker container, unlocked using the generated initial admin password, and configured through the setup wizard with the suggested plugins. A simple test job is then created and executed to verify that Jenkins can run automated builds.

## What is Continuous Integration?

Continuous Integration (CI) is a development practice where developers frequently integrate code changes into a shared repository. Each change triggers an automated build and test process, helping teams detect errors early and improve software quality.

### Key Features of CI

* Frequent code integration
* Automated builds
* Automated testing
* Immediate feedback to developers

### Benefits of CI

* Early bug detection
* Improved collaboration
* Faster development cycles
* Higher code quality

## Architecture

```text
┌─────────────────────┐
│      Developer      │
│    Pushes Code      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   Version Control   │
│     (Git Repo)      │
└──────────┬──────────┘
           │ Triggers
           ▼
┌─────────────────────┐
│   Jenkins Server    │
│  (Docker container) │
│   localhost:8080    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Build → Test       │
│  → Feedback         │
└─────────────────────┘
```

## Technologies Used

* **Jenkins** – Open-source automation server for CI/CD
* **Docker** – Runs Jenkins in a container
* **PowerShell** – Used to run the Docker commands
* **Web Browser** – Used to access the Jenkins dashboard

## Project Structure

```text
7-Jenkins-CI-Automation/
│
├── screenshots/
└── README.md
```

> No application code is required for this exercise. Jenkins is installed and managed entirely through Docker commands and the web interface.

## Prerequisites

Make sure the following are available:

* Docker Desktop installed and running
* Ports `8080` and `50000` free
* Internet access (to pull the Jenkins image and plugins)

Verify Docker:

```powershell
docker --version
```

## 1. Run Jenkins in Docker

Run:

```powershell
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
```

| Option                  | Purpose                                 |
| ----------------------- | --------------------------------------- |
| `-d`                    | Runs the container in the background    |
| `--name jenkins`        | Names the container                     |
| `-p 8080:8080`          | Jenkins web interface (external access) |
| `-p 50000:50000`        | Jenkins agent communication             |
| `jenkins/jenkins:lts`   | Official Jenkins LTS image              |

Example output:

```text
Unable to find image 'jenkins/jenkins:lts' locally
lts: Pulling from jenkins/jenkins
...
Status: Downloaded newer image for jenkins/jenkins:lts
<container-id>
```

If the container name is already in use, remove the old container and run the command again:

```powershell
docker rm -f jenkins
```

## 2. Verify the Container

```powershell
docker ps
```

The `jenkins` container should show:

```text
STATUS: Up
PORTS:  0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp
```

Jenkins takes about a minute to start. Check the logs if needed:

```powershell
docker logs jenkins
```

The log should contain:

```text
Jenkins is fully up and running
```

## 3. Get the Initial Admin Password

```powershell
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Example output:

```text
<32-character-password>
```

The password is also printed in the Jenkins logs.

## 4. Access Jenkins

Open:

```text
http://localhost:8080/
```

The **Unlock Jenkins** page is displayed.

## 5. Complete the Setup Wizard

```text
Unlock Jenkins
    → Install suggested plugins
    → Create First Admin User
    → Instance Configuration
    → Start using Jenkins
```

1. Paste the initial admin password and click **Continue**.
2. Select **Install suggested plugins** and wait for the installation to finish.
3. Create the first admin user.
4. Keep the default Jenkins URL: `http://localhost:8080/`.
5. Click **Save and Finish**, then **Start using Jenkins**.

The Jenkins dashboard is now available.

## 6. Run a Test Job

To verify that Jenkins can run builds:

```text
New Item
    → Name: test-job
    → Freestyle project
    → OK
```

Under **Build Steps**, select **Add build step → Execute shell** and enter:

```bash
echo "Hello from Jenkins CI"
```

Click **Save**, then **Build Now**.

Open the build and select **Console Output**:

```text
Started by user Admin
Running as SYSTEM
Building in workspace /var/jenkins_home/workspace/test-job
[test-job] $ /bin/sh -xe /tmp/jenkins<id>.sh
+ echo Hello from Jenkins CI
Hello from Jenkins CI
Finished: SUCCESS
```

## CI Workflow Demonstrated

```text
Trigger (Build Now)
       │
       ▼
Jenkins starts the build
       │
       ▼
Shell step runs
       │
       ▼
Result: SUCCESS / FAILURE
```

This follows the basic CI cycle: a change triggers a build, the build runs automated steps, and the developer receives feedback.

## Useful Commands

```powershell
docker ps                  # running containers
docker ps -a               # all containers
docker logs jenkins        # view Jenkins logs
docker stop jenkins        # stop Jenkins
docker start jenkins       # start Jenkins again
docker restart jenkins     # restart Jenkins
```

## Cleanup (Optional)

```powershell
docker stop jenkins
docker rm jenkins
docker rmi jenkins/jenkins:lts
```

> Only remove the container and image if the Jenkins setup is no longer needed.

## Expected Results

After completing the project, the following should be available:

* Jenkins running in a Docker container
* Jenkins web interface at `localhost:8080`
* Setup wizard completed with suggested plugins installed
* Admin user created
* Jenkins dashboard accessible
* `test-job` build finishing with `SUCCESS`

## Learning Outcomes

Through this project, the following DevOps concepts are demonstrated:

* Continuous Integration concepts
* Running Jenkins with Docker
* Container port mapping
* Retrieving the Jenkins initial admin password
* Jenkins setup wizard and plugin installation
* Creating and running a Jenkins job
* Reading Jenkins console output