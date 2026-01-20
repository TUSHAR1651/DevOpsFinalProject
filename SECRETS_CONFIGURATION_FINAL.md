# Secrets Configuration Guide

This document explains how to configure the required GitHub secrets for the DevOps CI/CD project.

## Required GitHub Secrets

### Overview
The project requires the following GitHub secrets to be configured in your repository:

| Secret Name | Purpose | Where Used |
|-------------|---------|------------|
| `DOCKERHUB_USERNAME` | Docker registry username | CI/CD pipelines |
| `DOCKERHUB_TOKEN` | Docker Hub access token | CI/CD pipelines |
| `KUBE_CONFIG` | Kubernetes cluster configuration | CD pipeline |

## Secret Configuration Steps

### 1. Navigate to GitHub Secrets

1. Go to your GitHub repository: https://github.com/TUSHAR1651/DevOpsFinalProject
2. Click on **Settings** tab
3. In the left sidebar, click **Secrets and variables** > **Actions**
4. Click **New repository secret**

### 2. Configure DOCKERHUB_USERNAME

#### Purpose
- Username for Docker Hub registry
- Used for logging in and pushing/pulling images

#### How to Get Docker Hub Username
1. Go to https://hub.docker.com/
2. Sign in to your Docker Hub account
3. Your username is displayed in the top-right corner
4. If you don't have an account, click **Sign Up**

#### Configuration
- **Name**: `DOCKERHUB_USERNAME`
- **Value**: Your Docker Hub username
- **Description**: Docker Hub registry username

### 3. Configure DOCKERHUB_TOKEN

#### Purpose
- Access token for Docker Hub registry
- Used for secure authentication in CI/CD pipelines
- Never hardcoded in workflows

#### How to Generate Docker Hub Token
1. Go to https://hub.docker.com/
2. Sign in to your Docker Hub account
3. Click on your profile icon → **Account Settings**
4. Scroll down to **Security** section
5. Click **New Access Token**
6. Fill in the form:
   - **Description**: `GitHub Actions CI/CD`
   - **Access permissions**: Read, Write, Delete
7. Click **Generate**
8. **IMPORTANT**: Copy the token immediately - you won't see it again

#### Configuration
- **Name**: `DOCKERHUB_TOKEN`
- **Value**: The access token you just generated
- **Description**: Docker Hub access token for CI/CD

### 4. Configure KUBE_CONFIG

#### Purpose
- Base64 encoded Kubernetes configuration file
- Used for CD pipeline to connect to Kubernetes cluster
- Contains cluster connection details and certificates

#### Option A: Local Kubernetes (minikube)

##### For Local Testing with minikube
1. Install minikube:
   ```bash
   # macOS
   brew install minikube
   
   # Linux
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube
   ```

2. Start minikube:
   ```bash
   minikube start --driver=docker
   ```

3. Get kubeconfig:
   ```bash
   cat ~/.kube/config | base64 -w 0
   ```

4. Copy the base64 output and use as KUBE_CONFIG secret value

#### Option B: Cloud Kubernetes Cluster

##### For Cloud Provider (AWS EKS, GCP GKE, Azure AKS)

1. **AWS EKS**:
   ```bash
   aws eks update-kubeconfig --name your-cluster-name
   cat ~/.kube/config | base64 -w 0
   ```

2. **GCP GKE**:
   ```bash
   gcloud container clusters get-credentials your-cluster-name --zone your-zone
   cat ~/.kube/config | base64 -w 0
   ```

3. **Azure AKS**:
   ```bash
   az aks get-credentials --resource-group your-rg --name your-cluster
   cat ~/.kube/config | base64 -w 0
   ```

#### Configuration
- **Name**: `KUBE_CONFIG`
- **Value**: The base64 encoded kubeconfig content
- **Description**: Base64 encoded Kubernetes configuration

## Security Best Practices

### Never Hardcode Secrets
- ❌ **WRONG**: `docker login -u myuser -p mytoken`
- ✅ **CORRECT**: `docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}`

### Secret Rotation
- **Docker Hub tokens**: Rotate every 90 days
- **Kubernetes configs**: Update when cluster changes
- **Review access**: Regularly audit who has access

### Access Control
- **Repository permissions**: Limit who can modify secrets
- **Environment-specific**: Use different secrets for different environments
- **Audit logging**: Monitor secret usage in GitHub Actions

## Verification

### Test CI Pipeline
1. Push a change to trigger CI pipeline
2. Check GitHub Actions tab for workflow runs
3. Verify Docker Hub authentication works
4. Confirm image is pushed successfully

### Test CD Pipeline
1. Create a git tag: `git tag v1.0.0`
2. Push the tag: `git push origin v1.0.0`
3. Check CD workflow execution
4. Verify Kubernetes deployment succeeds

## Troubleshooting

### Common Issues

#### Docker Hub Authentication Failed
- **Error**: `unauthorized: authentication required`
- **Solution**: Verify DOCKERHUB_USERNAME and DOCKERHUB_TOKEN are correct
- **Check**: Token has proper permissions (Read, Write, Delete)

#### Kubernetes Connection Failed
- **Error**: `unable to recognize the provider`
- **Solution**: Verify KUBE_CONFIG is properly base64 encoded
- **Check**: kubeconfig file is valid and not corrupted

#### Secret Not Found
- **Error**: `secrets.DOCKERHUB_USERNAME not found`
- **Solution**: Ensure secrets are configured in correct repository
- **Check**: Secret names match exactly (case-sensitive)

#### Permission Denied
- **Error**: `permission denied while accessing secrets`
- **Solution**: Check repository permissions
- **Verify**: GitHub Actions has access to secrets

### Debug Commands

#### Test Docker Hub Access
```bash
# Test Docker Hub authentication
echo "${DOCKERHUB_TOKEN}" | docker login -u "${DOCKERHUB_USERNAME}" --password-stdin
docker pull alpine
```

#### Test Kubernetes Access
```bash
# Test Kubernetes configuration
echo "${KUBE_CONFIG}" | base64 -d > test-kubeconfig
export KUBECONFIG=test-kubeconfig
kubectl cluster-info
```

## Environment-Specific Secrets

### Development Environment
- `DOCKERHUB_USERNAME_DEV`
- `DOCKERHUB_TOKEN_DEV`
- `KUBE_CONFIG_DEV`

### Staging Environment
- `DOCKERHUB_USERNAME_STAGING`
- `DOCKERHUB_TOKEN_STAGING`
- `KUBE_CONFIG_STAGING`

### Production Environment
- `DOCKERHUB_USERNAME_PROD`
- `DOCKERHUB_TOKEN_PROD`
- `KUBE_CONFIG_PROD`

## Automation

### GitHub Actions Usage
```yaml
# Example of using secrets in workflow
- name: Login to DockerHub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

- name: Configure kubectl
  run: |
    echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > kubeconfig
    export KUBECONFIG=kubeconfig
```

### Local Development
```bash
# Set environment variables for local testing
export DOCKERHUB_USERNAME=your_username
export DOCKERHUB_TOKEN=your_token
export KUBE_CONFIG=$(cat ~/.kube/config | base64 -w 0)
```

## Compliance

### Audit Requirements
- **Secret Inventory**: Maintain list of all secrets
- **Access Review**: Regular review of who has access
- **Rotation Schedule**: Automated secret rotation
- **Documentation**: Keep this guide updated

### Security Standards
- **Encryption**: All secrets encrypted at rest
- **Access Logging**: All secret access logged
- **Least Privilege**: Minimum required permissions
- **Regular Audits**: Quarterly security reviews

## Support

### Getting Help
1. **GitHub Documentation**: https://docs.github.com/en/actions/security-guides/using-secrets
2. **Docker Hub Support**: https://docs.docker.com/docker-hub/access-tokens/
3. **Kubernetes Documentation**: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/

### Emergency Procedures
1. **Immediate Rotation**: If secret is compromised
2. **Access Revocation**: Revoke old tokens immediately
3. **Password Changes**: Update all related credentials
4. **Incident Report**: Document security incidents

---

**Important**: Never commit secrets to your repository. Always use GitHub secrets for sensitive information.
