# 📘 Jenkins + AWS ECR CI Pipeline — 2025-06-02

## ✅ Objective

- Automate Docker image build and push to **AWS Elastic Container Registry (ECR)** using **Jenkins Pipeline**.
- Trigger on code commits and include **cleanup**, **success/failure logs**, and best practices.

---

## 🧱 Prerequisites

- ✅ AWS Account with ECR repositories created (`meanstack/frontend`, `meanstack/backend`)
- ✅ Jenkins installed with:
  - Docker and AWS CLI installed on Jenkins node
  - Valid AWS credentials added in Jenkins (`AWS-CRED`)
  - Git plugin enabled
- ✅ Source code (frontend/backend) pushed to GitHub

---

## 🧪 Pipeline Overview

```groovy
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '76956xxxxxxxx'
        ECR_REPO_F = 'meanstack/frontend'
        ECR_REPO_B = 'meanstack/backend'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {
        ...
    }

    post {
        ...
    }
}
```

---

## 🔁 Stages Explained

### 1. 🔄 Checkout Code

```groovy
git url: "https://github.com/kapilgole1/Mean-stack.git", branch: "main"
```

- Clones the latest code from the GitHub repo (`main` branch)
- Required before building Docker images

---

### 2. 🛠️ Build Docker Images

```groovy
sh "docker build -t ${ECR_REGISTRY}/${ECR_REPO_F}:${IMAGE_TAG} ./frontend"
sh "docker build -t ${ECR_REGISTRY}/${ECR_REPO_B}:${IMAGE_TAG} ./backend"
```

- Builds Docker images for frontend and backend using their respective `Dockerfile`
- Tags images with format:  
  `<account>.dkr.ecr.<region>.amazonaws.com/<repo>:<build_number>`

---

### 3. 🔐 Authenticate to AWS ECR

```groovy
withCredentials([...]) {
    sh '''
        aws ecr get-login-password --region $AWS_REGION | \
        docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
    '''
}
```

- Uses Jenkins credentials (`AWS-CRED`) to log in to AWS CLI
- Retrieves ECR login token and logs Docker into AWS ECR

---

### 4. 🚀 Push Images to ECR

```groovy
docker push ${ECR_REGISTRY}/${ECR_REPO_F}:${IMAGE_TAG}
docker push ${ECR_REGISTRY}/${ECR_REPO_B}:${IMAGE_TAG}
```

- Pushes both frontend and backend images to their respective ECR repositories

---

## 🧹 Post Section — Always Runs

```groovy
post {
    success {
        echo '✅ Pipeline completed successfully!'
    }

    failure {
        echo '❌ Pipeline failed. Please check logs above.'
    }

    always {
        sh 'docker image prune -f'
    }
}
```

| Block      | Purpose                                   |
|------------|-------------------------------------------|
| `success`  | Prints success message                    |
| `failure`  | Prints failure message                    |
| `always`   | Cleans up unused Docker images from agent |

---

## 💡 Key Learnings

| ✅ Task                                | 🚀 Learned |
|----------------------------------------|------------|
| Build and tag Docker images            | ✔️         |
| Authenticate Jenkins to AWS ECR        | ✔️         |
| Push images from Jenkins to ECR        | ✔️         |
| Use `environment` variables            | ✔️         |
| Use `post` block for cleanup and logs  | ✔️         |
| Handle AWS credentials in pipeline     | ✔️         |

---

## 📌 Next Steps (Optional)

- [ ] Trigger Jenkins pipeline automatically on GitHub commit (via webhook)
- [ ] Deploy pushed images to EKS or EC2
- [ ] Set up Slack/email alerts for build status
- [ ] Implement IaC (Terraform) for ECR provisioning
