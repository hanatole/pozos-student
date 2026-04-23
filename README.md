# Pozos Student API

This document describes step-by-step how to build the Docker image and run the application using Docker Compose.

---

## Dockerfile

```Dockerfile
FROM python:3.13-slim

LABEL maintainer="ash"

RUN apt update -y \
    && apt install -y gcc python3-dev libsasl2-dev libldap2-dev libssl-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt

COPY student_age.py .

RUN mkdir /data
VOLUME ["/data"]

EXPOSE 5000

CMD ["python3", "./student_age.py"]
```

---

## Building and Publishing the Docker Image

First, log in to the container registry. Then build the Docker image, tag it according to the chosen registry, and push the image to the registry.

In the example below, **Docker Hub** is used as the registry.

```bash
cd ./simple_api

echo "$REGISTRY_PASSWORD" | docker login -u ahagbes --password-stdin

docker build -t ahagbes/students:v0.1.0 .

docker push ahagbes/students:v0.1.0
```

---

## Runtime Environment

We use **Docker Compose** to orchestrate the runtime environment.

In the `frontend` service, update the `$url` variable so that it points to the API endpoint.

```php
# index.php
$url = 'http://api:5000/pozos/api/v1.0/get_student_ages';
```

---

## Environment Variables

Create a `.env` file to store the application credentials.

```env
# .env
USERNAME=
PASSWORD=
```

---

## Docker Compose Configuration

The Compose file orchestrates both the frontend and backend services.

```yaml
# compose.yml
services:

  website:
    image: php:apache
    container_name: website
    environment:
      USERNAME: ${USERNAME}
      PASSWORD: ${PASSWORD}
    volumes:
      - ./website:/var/www/html
    ports:
      - "8000:80"
    depends_on:
      - api

  api:
    image: ahagbes/students:v0.1.0
    container_name: api
    hostname: api
    volumes:
      - ./simple_api/student_age.json:/data/student_age.json
```

---

## Runtime Environment Structure

The environment directory structure should look like this:

```plaintext
.
├── .env
├── compose.yml
├── simple_api
│   └── student_age.json
└── website
    └── index.php
```

---

## Running the Application

To start the application stack, run:

```bash
docker compose up -d
```

This command launches both the API and the website services in detached mode.

---

## Demo

```html
<video controls width="600">
  <source src="./demo.webm" type="video/webm">
  Your browser does not support the video tag.
</video>
```
