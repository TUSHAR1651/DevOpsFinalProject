# How to Run Locally

This guide provides step-by-step instructions to run the DevOps Final Project locally on your machine.

## Prerequisites

### Required Software
- **Java 11** or higher
- **Maven 3.6+** 
- **Docker** and Docker Compose
- **Git** for version control
- **kubectl** for Kubernetes (optional)
- **minikube** for local Kubernetes (optional)

### System Requirements
- **RAM**: 4GB minimum, 8GB recommended
- **Storage**: 10GB free space
- **OS**: macOS, Linux, or Windows with WSL2

## Local Development Setup

### 1. Clone the Repository
```bash
git clone https://github.com/TUSHAR1651/DevOpsFinalProject.git
cd DevOpsFinalProject
```

### 2. Build the Application
```bash
# Using system Maven
mvn clean compile

# View build results
ls -la target/
```

### 3. Run Unit Tests
```bash
# Run all tests
mvn test

# View test results
ls -la target/surefire-reports/
```

### 4. Run Code Quality Checks
```bash
# Run Checkstyle
mvn checkstyle:check

# View Checkstyle report
ls -la target/checkstyle-results.html
```

### 5. Run Security Scans
```bash
# OWASP Dependency Check (requires NVD_API_KEY environment variable)
export NVD_API_KEY=your_api_key_here
mvn dependency-check:check

# View security report
ls -la target/dependency-check-report.html
```

## Running the Application

### Option 1: Maven Spring Boot Run
```bash
# Run using Maven
mvn spring-boot:run

# The application will start on http://localhost:8080
```

### Option 2: Build and Run JAR
```bash
# Build the application
mvn clean package -DskipTests

# Run the JAR file
java -jar target/demo-0.0.1-SNAPSHOT.jar

# The application will start on http://localhost:8080
```

### Option 3: Docker Container
```bash
# Build Docker image
docker build -t devops-demo:latest .

# Run container
docker run -d -p 8080:8080 --name devops-demo-local devops-demo:latest

# View logs
docker logs devops-demo-local

# Stop container
docker stop devops-demo-local
docker rm devops-demo-local
```

## Testing the Application

### Health Endpoints
```bash
# Basic health check
curl http://localhost:8080/health

# Spring Boot Actuator health
curl http://localhost:8080/actuator/health

# Application info
curl http://localhost:8080/actuator/info
```

### Expected Responses
```bash
# Health endpoint response
Application is healthy

# Actuator health response
{"status":"UP","groups":["liveness","readiness"]}
```

## Local Kubernetes Setup

### 1. Install minikube
```bash
# macOS
brew install minikube

# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Windows
choco install minikube
```

### 2. Install kubectl
```bash
# macOS
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### 3. Start minikube
```bash
# Start minikube
minikube start --driver=docker

# Verify status
minikube status
kubectl cluster-info
```

### 4. Deploy Application
```bash
# Load Docker image into minikube
minikube image load devops-demo:latest

# Apply Kubernetes manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Check deployment status
kubectl get pods
kubectl get services
```

### 5. Access Application
```bash
# Port forward to local
kubectl port-forward service/devops-demo-service 8080:80

# Or get minikube IP
minikube ip
# Access via http://$(minikube ip):8080
```

## Environment Configuration

### Application Properties
The application uses Spring Boot with default configuration:

#### Default Configuration
- **Server Port**: 8080
- **Management Endpoints**: /actuator/health, /actuator/info
- **Logging**: Default Spring Boot logging

### Environment Variables
```bash
# For OWASP Dependency Check
export NVD_API_KEY=your_nvd_api_key

# For Docker Hub access
export DOCKERHUB_USERNAME=your_username
export DOCKERHUB_TOKEN=your_token

# For Kubernetes access
export KUBECONFIG=path/to/kubeconfig
```

## Troubleshooting

### Common Issues

#### Maven Build Failures
```bash
# Clean and rebuild
mvn clean compile

# Check Java version
java -version
# Should be Java 11+

# Clear Maven cache
mvn dependency:purge-local-repository
```

#### Docker Build Issues
```bash
# Check Docker status
docker version
docker info

# Clean Docker cache
docker system prune -a

# Build with no cache
docker build --no-cache -t devops-demo:latest .
```

#### Port Conflicts
```bash
# Check what's using port 8080
lsof -i :8080

# Kill process
kill -9 <PID>

# Or use different port
docker run -d -p 9090:8080 devops-demo:latest
```

#### minikube Issues
```bash
# Restart minikube
minikube stop
minikube start

# Reset minikube
minikube delete
minikube start

# Check minikube logs
minikube logs
```

### Debug Mode

#### Enable Debug Logging
```bash
# Run with debug profile
mvn spring-boot:run -Dspring-boot.run.profiles=debug

# Or set environment variable
export SPRING_PROFILES_ACTIVE=debug
mvn spring-boot:run
```

## Performance Testing

### Basic Load Test
```bash
# Install Apache Bench (ab)
brew install ab  # macOS
sudo apt-get install apache2-utils  # Linux

# Run load test
ab -n 1000 -c 10 http://localhost:8080/health
```

### Stress Testing with curl
```bash
# Concurrent requests
for i in {1..20}; do
  curl -s http://localhost:8080/health &
done
wait
```

## Security Testing

### Local Security Scans
```bash
# Scan Docker image locally
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image devops-demo:latest
```

### Manual Security Tests
```bash
# Test for SQL injection
curl "http://localhost:8080/health?id=1' OR '1'='1"

# Test for XSS
curl "http://localhost:8080/health?name=<script>alert('xss')</script>"

# Check security headers
curl -I http://localhost:8080/health
```

## Cleanup

### Stop Application
```bash
# Stop Maven run
Ctrl+C

# Stop Docker container
docker stop devops-demo-local
docker rm devops-demo-local

# Stop Kubernetes deployment
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
```

### Clean Build Artifacts
```bash
# Maven cleanup
mvn clean

# Docker cleanup
docker system prune -a

# minikube cleanup
minikube delete
```

## Next Steps

After successful local setup:

1. **Test all endpoints** to verify functionality
2. **Run security scans** to check for vulnerabilities
3. **Test Kubernetes deployment** for container orchestration
4. **Review logs** for any issues
5. **Commit changes** and push to trigger CI pipeline

## Project Structure

```
DevOpsFinalProject/
├── .github/workflows/
│   ├── ci.yml              # CI pipeline
│   └── cd.yml              # CD pipeline
├── src/
│   ├── main/java/com/devops/demo/
│   └── test/
├── deployment.yaml            # Kubernetes deployment
├── service.yaml              # Kubernetes service
├── Dockerfile               # Docker configuration
├── pom.xml                 # Maven configuration
├── checkstyle.xml           # Code quality rules
└── README.md               # Project documentation
```

## Support

For issues:
1. Check the [troubleshooting section](#troubleshooting)
2. Review application logs
3. Check GitHub Actions workflow runs
4. Refer to the main [README.md](README.md)

---

**Happy Coding! 🚀**
