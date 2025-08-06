# Simple Node.js CI/CD Pipeline with Jenkins

This project demonstrates a complete CI/CD pipeline using Jenkins and Docker.

## Features
- Node.js web application
- Automated build and test
- Docker containerization
- Automated deployment
- Health checks

## Pipeline Stages
1. **Checkout**: Clone source code
2. **Build**: Install dependencies
3. **Test**: Run application tests
4. **Docker Build**: Create container image
5. **Deploy**: Deploy to target environment
6. **Verify**: Health check deployment

## Local Development
```bash
npm install
npm start
# Visit http://localhost:3000
