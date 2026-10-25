# DevOps Flask Project

A Python Flask application containerized using Docker.

## Features
- Home endpoint
- Health-check endpoint
- Automated tests with pytest
- Docker image and container deployment

## Run locally
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python app/app.py

## Run tests
python -m pytest -v

## Run with Docker
docker build -t devops-flask-app:v1 .
docker run -p 5000:5000 devops-flask-app:v1
