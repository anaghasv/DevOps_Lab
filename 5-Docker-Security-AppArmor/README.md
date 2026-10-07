# Docker Security with AppArmor

A hands-on DevOps exercise demonstrating how to secure a Dockerized Flask application using **AppArmor**, Docker security options, and the Docker Python SDK.

## Overview

This exercise demonstrates:

* Containerizing a Flask application using Docker
* Creating an AppArmor security profile
* Applying an AppArmor profile to a Docker container
* Automating Docker container management using Python
* Testing restricted actions inside a secured container

## Project Structure

```text
5-Docker-Security-AppArmor/
│
├── app.py
├── Dockerfile
├── my-apparmor-profile
├── apply_apparmor.py
├── test_restricted_actions.py
└── README.md
```

## Prerequisites

* Docker Desktop
* WSL2 / Ubuntu
* Python 3
* AppArmor utilities
* Docker Python SDK

## Task 1: Create the Flask Application

Create `app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, this is a secure Flask application running inside a Docker container!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

The application runs on port `5000`.

## Task 2: Create the Docker Image

Create a `Dockerfile`:

```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.8-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container
COPY . /app

# Install Flask
RUN pip install flask

# Expose port 5000
EXPOSE 5000

# Run app.py when the container launches
CMD ["python", "app.py"]
```

### Build the Docker image

```bash
docker build -t flask-apparmor .
```

### Run the container

```bash
docker run -d --name flask-test -p 5000:5000 flask-apparmor
```

Check the running container:

```bash
docker ps
```

The Flask application can be accessed at:

```text
http://localhost:5000
```

## Task 3: Create the AppArmor Profile

Create the file `my-apparmor-profile`:

```text
#include <tunables/global>

/usr/bin/python3 {
    # Deny access to sensitive system files
    deny /etc/** r,
    deny /var/** rw,

    # Allow Flask app to bind to port 5000
    network inet stream,

    # Permissions to the application directory
    /app/** rwk,

    # Deny execution of binaries
    deny /bin/** rmix,
    deny /usr/bin/** rmix,

    # Capability restrictions
    capability net_bind_service,
    deny capability sys_admin,
}
```

Load the AppArmor profile:

```bash
sudo apparmor_parser -r /etc/apparmor.d/my-apparmor-profile
```

Run the Docker container with the AppArmor profile:

```bash
docker run -d \
  --name apparmor-test \
  --security-opt="apparmor=my-apparmor-profile" \
  -p 5000:5000 \
  flask-apparmor
```

## Task 4: Apply AppArmor Using Python

Create a Python virtual environment:

```bash
python3 -m venv ~/docker-venv
```

Activate the environment:

```bash
source ~/docker-venv/bin/activate
```

Install the Docker Python SDK:

```bash
pip install docker
```

Create `apply_apparmor.py`:

```python
import docker

# Create a Docker client
client = docker.from_env()

# Build the Docker image
client.images.build(path=".", tag="flask-apparmor")

# Run the container with the AppArmor profile
container = client.containers.run(
    "flask-apparmor",
    ports={'5000/tcp': 5000},
    security_opt=["apparmor=my-apparmor-profile"],
    detach=True
)

print(f"Container started: {container.short_id}")

# Verify AppArmor profile applied
container_info = client.api.inspect_container(container.id)
apparmor_profile = container_info['HostConfig']['SecurityOpt']

print(f"AppArmor profile applied: {apparmor_profile}")

# Stop the container
container.stop()
```

Run:

```bash
python apply_apparmor.py
```

The script builds the Docker image, starts a container with the AppArmor profile, verifies the security option, and stops the container.

## Task 5: Test Restricted Actions

Create `test_restricted_actions.py`:

```python
import docker

# Create a Docker client
client = docker.from_env()

# Run the container with the AppArmor profile
container = client.containers.run(
    "flask-apparmor",
    ports={'5000/tcp': 5000},
    security_opt=["apparmor=my-apparmor-profile"],
    detach=True
)

# Test restricted actions
exit_code, output = container.exec_run("cat /etc/passwd")
print(f"Attempt to read /etc/passwd: Exit Code {exit_code}, Output: {output.decode()}")

exit_code, output = container.exec_run("/bin/bash")
print(f"Attempt to execute /bin/bash: Exit Code {exit_code}, Output: {output.decode()}")

# Stop the container
container.stop()
```

Run:

```bash
python test_restricted_actions.py
```

The script tests whether the AppArmor profile restricts:

* Reading `/etc/passwd`
* Executing `/bin/bash`

### Expected Output

```text
Attempt to read /etc/passwd: Exit Code 1
Attempt to execute /bin/bash: Exit Code 126
```

## Key Learning Outcomes

Through this exercise, the following concepts are demonstrated:

1. Docker containerization of a Flask application.
2. Creating and configuring an AppArmor security profile.
3. Applying AppArmor profiles to Docker containers.
4. Using Docker security options.
5. Automating Docker operations using Python.
6. Testing container security restrictions.
7. Using Linux security mechanisms to improve container isolation.

## Conclusion

This exercise demonstrates how **Docker and AppArmor** can be used together to provide additional security for containerized applications. The Flask application is containerized, an AppArmor profile is configured, and Python scripts are used to automate and test the security configuration.
