Running a Simple Multi-Container Application with Docker Compose

```
mkdir docker-compose-lab && cd docker-compose-lab
mkdir app
```
Create the flask application
```
vi app/app.py
```
Copy paste the contents:
```
from flask import Flask
import redis

app = Flask(__name__)
cache = redis.StrictRedis(host='redis', port=6379, decode_responses=True)

@app.route('/')
def hello():
    count = cache.incr('hits')
    return f"Hello World! You've visited {count} times."

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
Save and Exit.

Create a requirements.txt
```
vi app/requirements.txt
```
Copy paste the contents
```
flask
redis
```

Create a Dockerfile for Flask
```
vi app/Dockerfile
```
Copy paste the contents
```
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

Define the Docker Compose File
```
vi docker-compose.yml
```
Copy paste the contents
```
version: '3.9'
services:
  web:
    build: ./app
    ports:
      - "5000:5000"
    depends_on:
      - redis
  redis:
    image: redis:6.2
```

Run the Application
Build and Start Services:
```
docker-compose up --build
```
Access the Application: Open your browser and navigate to http://localhost:5000.
You’ll see a message:
Hello World! You've visited X times.

Stop the Application:
```
docker-compose down
```
