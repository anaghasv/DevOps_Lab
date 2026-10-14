# Jenkins Multi-Stage Pipeline for a Python Flask Application

## Overview

This project demonstrates Continuous Integration and basic Continuous Deployment (CI/CD) using Jenkins for a Python Flask application. The Jenkins pipeline automatically checks out the source code from GitHub, installs dependencies, runs unit tests, deploys the application files, starts the application, and performs an application test.

## Technologies Used

- **Jenkins** – CI/CD automation
- **Git and GitHub** – Version control and source code management
- **Python** – Application development
- **Flask** – Web application framework
- **unittest** – Unit testing
- **Linux Shell** – Pipeline commands

## Project Structure

```text
9-Jenkins-MultiStage-Pipeline/
├── app.py
├── requirements.txt
├── test_app.py
├── Jenkinsfile
└── README.md
```

## Pipeline Stages

### 1. Build
- Creates a Python virtual environment.
- Installs the required dependencies from `requirements.txt`.

### 2. Test
- Runs unit tests using Python's `unittest` framework.
- Verifies the application functionality.

### 3. Deploy
- Creates a deployment directory.
- Copies `app.py` and `requirements.txt` into the deployment directory.

### 4. Run Application
- Starts the Flask application as a background process.
- Redirects application output to a log file.
- Stores the process ID in `app.pid`.

### 5. Test Application
- Uses Flask's test client to send a GET request to the application's root route.
- Verifies that the response status code is `200`.
- Checks that the response message is `Hello, Jenkins Multi-Stage Pipeline!`.

## Jenkins Configuration

1. Create a new **Pipeline** job in Jenkins.
2. Select **Pipeline script from SCM** as the definition.
3. Select **Git** as the source control management system.
4. Enter the GitHub repository URL:

   `https://github.com/anaghasv/DevOps_Lab`

5. Set the branch specifier to `*/main`.
6. Set the Script Path to:

   `9-Jenkins-MultiStage-Pipeline/Jenkinsfile`

7. Save the configuration and click **Build Now**.

## Dependencies

The application dependencies are specified in `requirements.txt`:

```text
flask==2.1.2
Werkzeug==2.2.2
```

Python 3, `pip`, and `venv` must be available in the Jenkins build environment.

## Expected Output

When the pipeline completes successfully, Jenkins should show:

- Build — Success
- Test — Success
- Deploy — Success
- Run Application — Success
- Test Application — Success

The console output should include:

```text
Ran 1 test ... OK
Application test passed!
Pipeline completed successfully!
Finished: SUCCESS
```

## Learning Outcomes

- Creating a multi-stage Jenkins pipeline using a `Jenkinsfile`.
- Integrating Jenkins with a GitHub repository.
- Automating Python dependency installation and unit testing.
- Performing basic deployment through shell commands.
- Running a Flask application from a Jenkins pipeline.
- Understanding automated validation and post-build actions.

## Note

The current application test uses Flask's test client to verify the application route. It does not independently verify that the background application process is accessible over the network.

