# Django Docker Deployment Guide

## Understanding the Dockerfile

A Dockerfile is like a recipe that tells Docker how to build your application container. Let's break down each part:

### 1. Base Image
```dockerfile
FROM python:3.10-slim
```
- **What it does**: Starts with a pre-built Python 3.10 image
- **Why slim**: Smaller size, faster downloads, fewer security vulnerabilities
- **Think of it as**: Choosing your operating system foundation

### 2. Environment Variables
```dockerfile
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
```
- **PYTHONDONTWRITEBYTECODE 1**: Prevents Python from creating .pyc files (compiled bytecode)
- **PYTHONUNBUFFERED 1**: Makes Python output appear immediately in logs
- **Why important**: Better debugging and cleaner container

### 3. Working Directory
```dockerfile
WORKDIR /app
```
- **What it does**: Sets `/app` as the main folder inside the container
- **Think of it as**: Creating your project workspace

### 4. System Dependencies
```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        gcc \
    && rm -rf /var/lib/apt/lists/*
```
- **apt-get update**: Updates the package list
- **gcc**: Compiler needed for some Python packages
- **--no-install-recommends**: Installs only essential packages
- **rm -rf /var/lib/apt/lists/***: Cleans up to reduce image size

### 5. Python Dependencies
```dockerfile
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt
```
- **COPY requirements.txt**: Copies your Python package list into container
- **pip install**: Installs all Python packages
- **--no-cache-dir**: Doesn't store cache, saves space

### 6. Copy Application Code
```dockerfile
COPY . /app/
```
- **What it does**: Copies all your project files into the container
- **The dot (.)**: Means "everything in current directory"

### 7. Static Files
```dockerfile
RUN python manage.py collectstatic --noinput
```
- **collectstatic**: Gathers all CSS, JS, images into one folder
- **--noinput**: Runs without asking for confirmation
- **Why needed**: For serving static files in production

### 8. Network Port
```dockerfile
EXPOSE 8000
```
- **What it does**: Tells Docker this app uses port 8000
- **Note**: Doesn't actually open the port, just documents it

### 9. Start Command
```dockerfile
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```
- **CMD**: The command that runs when container starts
- **0.0.0.0:8000**: Makes the app accessible from outside the container
- **Why not 127.0.0.1**: That would only work inside the container

## Building Your Own Dockerfile

### Basic Structure Template:
```dockerfile
# 1. Choose base image
FROM python:3.10-slim

# 2. Set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# 3. Create working directory
WORKDIR /app

# 4. Install system packages (if needed)
RUN apt-get update && apt-get install -y package-name

# 5. Install Python dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# 6. Copy your code
COPY . .

# 7. Expose port
EXPOSE 8000

# 8. Define startup command
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### Common Customizations:

**For different Python versions:**
```dockerfile
FROM python:3.11-slim  # or 3.9-slim, 3.8-slim
```

**For additional system packages:**
```dockerfile
RUN apt-get update && apt-get install -y \
    postgresql-client \
    git \
    curl
```

**For production (with Gunicorn):**
```dockerfile
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myproject.wsgi:application"]
```

**For database migrations:**
```dockerfile
RUN python manage.py migrate
```

## Building and Running

### Build the image:
```bash
docker build -t my-django-app .
```

### Run the container:
```bash
docker run -p 8000:8000 my-django-app
```

### Access your app:
Open http://localhost:8000 in your browser

## Tips for Beginners

1. **Layer Order Matters**: Put things that change less often (like requirements.txt) before things that change often (your code)
2. **Use .dockerignore**: Like .gitignore, but for Docker
3. **Keep Images Small**: Use slim base images and clean up after installations
4. **One Process Per Container**: Don't run multiple services in one container
5. **Environment Variables**: Use them for configuration that changes between environments
