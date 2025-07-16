# Upload Modified Unmute Code to GitHub

## Steps to Upload to Your Repository

### 1. Navigate to the Project Directory
```bash
cd /Users/girpatil/Downloads/ClaudeCode/kimi/unmute
```

### 2. Initialize Git and Add Remote
```bash
# Initialize git repository
git init

# Add your GitHub repository as remote
git remote add origin https://github.com/girishp1983/unmute.git

# Fetch existing branches
git fetch origin
```

### 3. Create and Switch to New Branch
```bash
# Create new branch for Groq integration
git checkout -b groq-integration

# Or if you want to work on existing main branch
# git checkout -b groq-integration origin/main
```

### 4. Add and Commit All Changes
```bash
# Add all files
git add .

# Commit with descriptive message
git commit -m "Replace LLM with Groq API for moonshotai/kimi-k2-instruct

- Added groq dependency to pyproject.toml
- Created GroqStream class for moonshotai/kimi-k2-instruct model
- Updated unmute_handler.py to use GroqStream instead of VLLMStream
- Modified system_prompt.py to show new model name
- Updated example script to use GroqStream
- Added Kubernetes deployment manifests for AWS EKS
- Created deployment documentation for G6e.12xlarge instances
- Added Dockerfiles for backend and frontend deployment"
```

### 5. Push to GitHub
```bash
# Push new branch to your repository
git push -u origin groq-integration
```

### 6. Create Pull Request (Optional)
After pushing, you can create a pull request on GitHub to merge the changes into your main branch.

## Alternative: Direct Upload to Main Branch
If you want to upload directly to main branch:

```bash
# Switch to main branch
git checkout -b main origin/main

# Add and commit changes
git add .
git commit -m "Integrate Groq API for moonshotai/kimi-k2-instruct model"

# Push to main
git push origin main
```

## Files Modified/Added:
- `pyproject.toml` - Added groq dependency
- `unmute/llm/llm_utils.py` - Added GroqStream class
- `unmute/llm/system_prompt.py` - Updated model name
- `unmute/unmute_handler.py` - Updated to use GroqStream
- `unmute/scripts/vllm_wrapper_example.py` - Updated example
- `k8s/` - New directory with Kubernetes manifests
- `Dockerfile.backend` - Backend container definition
- `Dockerfile.frontend` - Frontend container definition
- `AWS_EKS_DEPLOYMENT.md` - Deployment documentation

## Important Notes:
- Make sure you have write access to the repository
- You may need to authenticate with GitHub (use personal access token)
- The original kyutai-labs/unmute repository will remain as upstream
- Your changes are specifically for Groq integration

## GitHub Authentication:
If you need to authenticate, use:
```bash
# For HTTPS (recommended)
git config --global credential.helper store

# Or use SSH keys
git remote set-url origin git@github.com:girishp1983/unmute.git
```