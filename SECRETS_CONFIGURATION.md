# Secrets Configuration Guide

## 🔐 Required GitHub Secrets

This document outlines the required secrets for the Advanced DevOps CI/CD project and provides step-by-step instructions for configuration.

### 1. DockerHub Credentials

#### DOCKERHUB_USERNAME
- **Purpose**: Docker Hub registry username for image push/pull operations
- **How to Obtain**: Your Docker Hub username
- **Configuration**: 
  1. Go to GitHub repository Settings → Secrets and variables → Actions
  2. Click "New repository secret"
  3. Name: `DOCKERHUB_USERNAME`
  4. Value: Your Docker Hub username

#### DOCKERHUB_TOKEN
- **Purpose**: Access token for secure Docker Hub operations
- **How to Obtain**: 
  1. Log in to Docker Hub
  2. Go to Account Settings → Security
  3. Click "New Access Token"
  4. Description: `GitHub Actions CI/CD`
  5. Permissions: Read, Write, Delete (or as needed)
  6. Copy the generated token
- **Configuration**:
  1. In GitHub repository, add new secret
  2. Name: `DOCKERHUB_TOKEN`
  3. Value: Paste the copied token

### 2. Security Scanning

#### NVD_API_KEY
- **Purpose**: API key for National Vulnerability Database dependency scanning
- **How to Obtain**:
  1. Go to [NVD API Key Request](https://nvd.nist.gov/developers/request-an-api-key)
  2. Fill out the request form with your organization details
  3. Wait for approval (typically 1-3 business days)
  4. Receive API key via email
- **Configuration**:
  1. Add new secret in GitHub repository
  2. Name: `NVD_API_KEY`
  3. Value: Your NVD API key

### 3. Kubernetes Deployment

#### KUBE_CONFIG
- **Purpose**: Base64 encoded Kubernetes configuration file for cluster access
- **How to Obtain**:
  1. Install and configure kubectl locally
  2. Set up cluster access (for cloud providers, use their CLI tools)
  3. Test connection: `kubectl cluster-info`
  4. Export config: `cat ~/.kube/config`
- **Configuration**:
  1. Encode the config file:
     ```bash
     cat ~/.kube/config | base64 -w 0
     ```
  2. Copy the base64 output
  3. Add new secret in GitHub repository
  4. Name: `KUBE_CONFIG`
  5. Value: Paste the base64 encoded config

## 🛠️ Step-by-Step Configuration

### Prerequisites
- GitHub repository admin access
- Docker Hub account
- Kubernetes cluster access
- NVD API key (for dependency scanning)

### Configuration Steps

#### Step 1: Docker Hub Setup
1. **Create Docker Hub Account** (if not already have one)
2. **Generate Access Token**:
   - Log in to Docker Hub
   - Navigate to Account Settings → Security
   - Click "New Access Token"
   - Set appropriate permissions
   - Copy the token immediately (won't be shown again)

#### Step 2: GitHub Secrets Configuration
1. **Navigate to Repository Settings**:
   - Go to your GitHub repository
   - Click "Settings" tab
   - Navigate to "Secrets and variables" → "Actions"

2. **Add Each Secret**:
   - Click "New repository secret"
   - Enter the name and value as specified above
   - Click "Add secret"

3. **Verify Secrets**:
   - Ensure all secrets are listed in the repository
   - Check for any typos or formatting issues

#### Step 3: Kubernetes Configuration
1. **Set up kubectl**:
   ```bash
   # Install kubectl
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/
   
   # Configure cluster access (example for EKS)
   aws eks update-kubeconfig --region <region> --name <cluster-name>
   ```

2. **Test Cluster Access**:
   ```bash
   kubectl cluster-info
   kubectl get nodes
   ```

3. **Create Docker Registry Secret**:
   ```bash
   kubectl create secret docker-registry dockerhub-secret \
     --docker-server=docker.io \
     --docker-username=<DOCKERHUB_USERNAME> \
     --docker-password=<DOCKERHUB_TOKEN> \
     --namespace=devops-demo
   ```

#### Step 4: NVD API Key Setup
1. **Request API Key**:
   - Visit [NVD Developers Portal](https://nvd.nist.gov/developers)
   - Submit API key request
   - Wait for approval email

2. **Configure in GitHub**:
   - Add `NVD_API_KEY` secret as described above

## 🔍 Testing Configuration

### Test CI Pipeline
1. **Push to main branch**:
   ```bash
   git add .
   git commit -m "Test CI pipeline"
   git push origin main
   ```

2. **Monitor GitHub Actions**:
   - Go to repository "Actions" tab
   - Check CI workflow execution
   - Verify all stages complete successfully

### Test CD Pipeline
1. **Create Git Tag**:
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

2. **Monitor Deployment**:
   - Check CD workflow execution
   - Verify Kubernetes deployment
   - Test application endpoints

## 🚨 Common Issues & Troubleshooting

### Docker Hub Issues
- **Authentication Failed**: Verify DOCKERHUB_TOKEN is correct and has proper permissions
- **Permission Denied**: Ensure token has write permissions for image pushing
- **Rate Limiting**: Check Docker Hub rate limits for your account

### Kubernetes Issues
- **Invalid KUBE_CONFIG**: Verify base64 encoding is correct
- **Cluster Access**: Ensure kubectl can connect to cluster locally
- **Namespace Issues**: Create devops-demo namespace if it doesn't exist

### NVD API Issues
- **Rate Limiting**: NVD API has rate limits; pipeline includes delays
- **Invalid API Key**: Ensure API key is correctly copied and not expired
- **Network Issues**: Check GitHub Actions runner network access

### General Troubleshooting
1. **Check Secret Names**: Ensure exact spelling and case
2. **Verify Values**: Double-check secret values for typos
3. **Review Logs**: Check GitHub Actions workflow logs for detailed errors
4. **Test Locally**: Replicate steps locally to isolate issues

## 📋 Security Best Practices

### Secret Management
- **Never Hardcode Secrets**: Always use GitHub repository secrets
- **Regular Rotation**: Rotate secrets periodically (especially tokens)
- **Principle of Least Privilege**: Grant minimum necessary permissions
- **Audit Access**: Monitor secret usage and access patterns

### Docker Hub Security
- **Use Access Tokens**: Don't use password for automation
- **Token Expiration**: Set appropriate expiration dates
- **Permission Scoping**: Limit token permissions to required operations

### Kubernetes Security
- **RBAC**: Use Role-Based Access Control
- **Namespace Isolation**: Deploy in dedicated namespaces
- **Network Policies**: Implement network security policies

## 🔄 Maintenance

### Regular Tasks
1. **Token Rotation**: Rotate Docker Hub tokens every 90 days
2. **API Key Renewal**: Monitor NVD API key expiration
3. **Cluster Credentials**: Update kubeconfig if cluster changes
4. **Permission Review**: Regularly review and update permissions

### Monitoring
1. **Pipeline Success Rates**: Monitor CI/CD execution success
2. **Security Scan Results**: Review vulnerability findings
3. **Resource Usage**: Monitor Kubernetes resource consumption
4. **Access Logs**: Review secret usage and access patterns

---

## 📞 Support

For issues with secrets configuration:
1. Check GitHub Actions workflow logs
2. Verify secret names and values
3. Test local connectivity to services
4. Review this documentation for common issues

**Important**: Never share secrets or commit them to version control. Always use secure secret management practices.
