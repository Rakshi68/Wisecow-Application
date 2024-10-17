# Wisecow-Application
Dockerfile Creation:
# Use an official Node.js runtime as a parent image
FROM node:14

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install app dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose the port the app runs on
EXPOSE 3000

# Command to run the application
CMD ["npm", "start"]
Kubernetes Manifest Files:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
      - name: wisecow
        image: your-dockerhub-username/wisecow:latest
        ports:
        - containerPort: 3000
apiVersion: v1
kind: Service
metadata:
  name: wisecow-service
spec:
  selector:
    app: wisecow
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
GitHub Actions Workflow:
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and Push Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: your-dockerhub-username/wisecow:latest

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Set up kubectl
        uses: azure/setup-kubectl@v1
        with:
          version: 'latest'

      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f wisecow-deployment.yaml
          kubectl apply -f wisecow-service.yaml


Part 2: Problem Statement 2 Solutions
Objective 1: System Health Monitoring Script
import psutil
import logging

# Configure logging
logging.basicConfig(filename='system_health.log', level=logging.INFO)

# Define thresholds
CPU_THRESHOLD = 80.0
MEMORY_THRESHOLD = 80.0
DISK_THRESHOLD = 80.0

def check_system_health():
    cpu_usage = psutil.cpu_percent()
    memory_usage = psutil.virtual_memory().percent
    disk_usage = psutil.disk_usage('/').percent

    if cpu_usage > CPU_THRESHOLD:
        logging.warning(f'CPU usage high: {cpu_usage}%')
    if memory_usage > MEMORY_THRESHOLD:
        logging.warning(f'Memory usage high: {memory_usage}%')
    if disk_usage > DISK_THRESHOLD:
        logging.warning(f'Disk usage high: {disk_usage}%')

    print("System health checked.")

if __name__ == "__main__":
    check_system_health()

Objective 2: Application Health Checker
import requests

def check_application_status(url):
    try:
        response = requests.get(url)
        if response.status_code == 200:
            print(f"Application is UP (Status code: {response.status_code})")
        else:
            print(f"Application is DOWN (Status code: {response.status_code})")
    except requests.exceptions.RequestException as e:
        print(f"Application is DOWN (Error: {e})")

if __name__ == "__main__":
    check_application_status('http://your-wisecow-service-url')
