# Simple Maven CI/CD Pipeline Project

This project demonstrates an end-to-end Jenkins pipeline that:

1. Pulls code from GitHub  
2. Builds a Maven Java app  
3. Builds a Docker image  
4. Pushes it to DockerHub  
5. Runs the container automatically  

## Setup
- Configure Jenkins with Git, Maven, and Docker
- Add credentials:
  - `github-cred` for GitHub (optional)
  - `dockerhub-cred` for DockerHub
- Enable GitHub Webhook → Jenkins URL: `/github-webhook/`
