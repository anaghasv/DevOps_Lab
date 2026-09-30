# Docker Networking with Multiple Containers

This exercise demonstrates Docker networking by running a Flask application, MySQL database, and Redis cache on the same custom Docker bridge network.

## Objective

* Create a custom Docker bridge network.
* Run multiple containers on the same network.
* Understand container-to-container communication.
* Test Docker DNS using container names.
* Access the Flask application from the host machine.

## Technologies Used

* Docker
* Python
* Flask
* MySQL
* Redis

## Files

```text
.
├── app.py
├── requirements.txt
├── Dockerfile
└── README.md
```

## Steps Followed

### 1. Checked Docker

Verified that Docker was installed and available:

```powershell
docker --version
```

### 2. Created a Custom Docker Network

Created a bridge network named `my-bridge-net`:

```powershell
docker network create --driver bridge my-bridge-net
```

### 3. Verified the Network

Listed the available Docker networks:

```powershell
docker network ls
```

Inspected the newly created network:

```powershell
docker network inspect my-bridge-net
```

The network used the Docker bridge driver with:

```text
Subnet: 172.18.0.0/16
Gateway: 172.18.0.1
```

### 4. Created the Flask Application

The Flask application provides an `/about` REST API endpoint.

`app.py`:

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/about', methods=['GET'])
def about():
    return jsonify({
        "name": "Simple REST API",
        "version": "1.0",
        "description": "This is a simple REST API built with Flask."
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True, port=5001)
```

### 5. Created `requirements.txt`

```text
Flask==2.0.1
Werkzeug==2.0.3
```

`Werkzeug==2.0.3` was specified to maintain compatibility with Flask 2.0.1.

### 6. Created the Dockerfile

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
COPY app.py .

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 5001

CMD ["python", "app.py"]
```

### 7. Built the Flask Docker Image

Built the Docker image:

```powershell
docker build -t flask-api .
```

### 8. Started the MySQL Container

Started MySQL and connected it to the custom network:

```powershell
docker run -d `
  --name mysql `
  --network my-bridge-net `
  -e MYSQL_ROOT_PASSWORD=rootpass `
  -e MYSQL_DATABASE=devopsdb `
  mysql:latest
```

### 9. Started the Redis Container

Started Redis on the same network:

```powershell
docker run -d `
  --name redis `
  --network my-bridge-net `
  redis:latest
```

### 10. Started the Flask Container

Started Flask on the same network and mapped port `5001`:

```powershell
docker run -d `
  --name flask `
  --network my-bridge-net `
  -p 5001:5001 `
  flask-api
```

### 11. Verified the Containers

Checked the running containers:

```powershell
docker ps
```

The following containers were running:

```text
flask
mysql
redis
```

### 12. Tested the Flask API

Opened the following URL in the browser:

```text
http://localhost:5001/about
```

The API returned:

```json
{
  "description": "This is a simple REST API built with Flask.",
  "name": "Simple REST API",
  "version": "1.0"
}
```

### 13. Tested Container-to-Container Communication

Entered the Flask container:

```powershell
docker exec -it flask bash
```

Since `ping` was not available in the `python:3.9-slim` image, Python DNS resolution was used to test connectivity.

Tested MySQL:

```bash
python -c "import socket; print(socket.gethostbyname('mysql'))"
```

Result:

```text
172.18.0.2
```

Tested Redis:

```bash
python -c "import socket; print(socket.gethostbyname('redis'))"
```

Result:

```text
172.18.0.3
```

This confirmed that the Flask container could resolve the MySQL and Redis containers using their container names.

Exited the Flask container:

```bash
exit
```

### 14. Inspected the Final Network

Ran:

```powershell
docker network inspect my-bridge-net
```

The network contained all three containers:

```text
mysql  → 172.18.0.2
redis  → 172.18.0.3
flask  → 172.18.0.4
```

This confirmed that Flask, MySQL, and Redis were connected to the same Docker bridge network.

### 15. Stopped the Containers

After completing the testing, the containers were stopped:

```powershell
docker stop flask mysql redis
```

## Result

The exercise successfully demonstrated:

* Creation of a custom Docker bridge network.
* Running Flask, MySQL, and Redis containers.
* Connecting multiple containers to the same network.
* Container-name-based DNS resolution.
* Flask API access through host port mapping.
* Inspection and management of Docker networks and containers.
