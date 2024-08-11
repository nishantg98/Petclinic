
# 🚀 Automated GitLab CI/CD Pipeline with Kubernetes

Welcome to the **Automated GitLab CI/CD Pipeline with Kubernetes** project, a robust and secure CI/CD pipeline implemented with GitLab and seamlessly integrated with Kubernetes. This pipeline is designed to automate the software development and deployment processes, ensuring efficiency, security, and scalability.

## 🎓 Project Overview
This project is inspired by real-time corporate GitLab CI/CD practices, as demonstrated in the **DevOps Shack** course. The pipeline follows a structured approach, with each stage focusing on specific tasks to streamline the development lifecycle. Below is a detailed breakdown of each stage:

![1720536088673](https://github.com/user-attachments/assets/3502e07c-5575-4ef6-9c58-184a30c5dff4)


### Stage 1: Install Required Tools
The first stage ensures that all necessary tools are installed on the CI/CD server.

```yaml
install_required_tools:
  stage: setup
  script:
    - sudo apt install openjdk-17-jre-headless -y
    - sudo apt install maven -y
    - sudo apt-get install wget apt-transport-https gnupg lsb-release
    - wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
    - echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
    - sudo apt-get update
    - sudo apt-get install trivy
    - sudo apt install docker.io -y && sudo chmod 666 /var/run/docker. sock
    - sudo snap install kubectl --classic
  tags:
    - self-hosted
```

**Explanation:**
- **Stage Name:** `install_tools`, part of the `setup` stage.
- **Purpose:** Installs essential tools like JDK, Maven, trivy, docker, kubectl.
- **Tags:** Runs on a runner with the `self-hosted` tag.


### Stage 2: Execute Test Cases
This stage involves running automated tests to validate the functionality of the code.

```yaml
execute_test_cases:
  stage: test
  script:
   - mvn test
  tags:
   - self-hosted
```

**Explanation:**
- **Stage Name:** `test`, part of the `test` stage.
- **Purpose:** Executes test cases using `Maven`.
- **Tags:** Runs on a runner with the `self-hosted` tag.

### Stage 3: Perform File System Scan (Trivy)
In this stage, project files are scanned for vulnerabilities using Trivy.

```yaml
perform_file_system_scan:
  stage: security
  script:
   - trivy fs --format table -o fs.html .
  tags:
   - self-hosted
```

**Explanation:**
- **Stage Name:** `file_system_scan`, part of the `security` stage.
- **Purpose:** Runs Trivy to scan the project directory for vulnerabilities.
- **Tags:** Runs on a runner with the `self-hosted` tag.

### Stage 4: Evaluate Code Quality (SonarQube)
This stage involves evaluating code quality using SonarQube's static analysis.

```yaml
sonarqube-check:
  stage: security  
  image: 
    name: sonarsource/sonar-scanner-cli:latest
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
    GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script: 
    - sonar-scanner
  allow_failure: true
  only:
    - main
```

**Explanation:**
- **Stage Name:** `code_quality_analysis`, part of the `security` stage.
- **Purpose:** Runs SonarQube's static analysis.
- **Tags:** Runs on a runner with the `self-hosted` tag.

![Screenshot from 2024-08-11 17-45-41](https://github.com/user-attachments/assets/5a6eb2e1-78e4-4216-af46-6e9def40f44f)


### Stage 5: Build Application Artifact
This stage involves compiling and packaging the application into an executable artifact.

```yaml
build_application_artifact:
  stage: build
  script:
    - mvn  package
  tags:
    - self-hosted
  only:
    - main
```

**Explanation:**
- **Stage Name:** `build_artifact`, part of the `build` stage.
- **Purpose:** Packages the application using Maven.
- **Tags:** Runs on a runner with the `self-hosted` tag.


### Stage 6: Build Docker Image
This stage creates a Docker image for containerized deployment.

```yaml
build_docker_image:
  stage: build
  script:
    - docker login -u $DOCKER_USER -p $DOCKER_PASS
    - mvn package  
    - docker build -t nishantg98/petclinic:latest .
  tags:
    - self-hosted
```

**Explanation:**
- **Stage Name:** `build_docker_image`, part of the `build` stage.
- **Purpose:** Builds a Docker image using the Dockerfile.
- **Tags:** Runs on a runner with the `self-hosted` tag.

### Stage 7: Scan Docker Image for Security Vulnerabilities (Trivy)
This stage checks the Docker image for security vulnerabilities using Trivy.

```yaml
scan_docker_image:
  stage: build
  script:
    - trivy image --format table -o trivy-image-report.html nishantg98/boardgame:latest 
  tags:
   - self-hosted
```

**Explanation:**
- **Stage Name:** `scan_docker_image`, part of the `build` stage.
- **Purpose:** Runs Trivy to scan the Docker image for vulnerabilities.
- **Tags:** Runs on a runner with the `self-hosted` tag.

### Stage 8: Push Docker Image to Docker Hub
This stage uploads the Docker image to Docker Hub for distribution.

```yaml
push_to_docker_hub:
  stage: build
  script:
    - docker login -u $DOCKER_USER -p $DOCKER_PASS
    - docker push nishantg98/petclinic:latest
  tags:
   - self-hosted
```

**Explanation:**
- **Stage Name:** `push_to_docker_hub`, part of the `build` stage.
- **Purpose:** Pushes the Docker image to Docker Hub.
- **Tags:** Runs on a runner with the `self-hosted` tag.

### Stage 9: Deploy Application to Kubernetes Cluster
This stage deploys the application to a Kubernetes cluster for orchestration.

```yaml
k8s-deploy:
  stage: deploy 
  variables:
    KUBECONFIG_PATH: /home/ubuntu/.kube/config
  before_script:
    - mkdir -p $(dirname "$KUBECONFIG PATH")
    - echo "$KUBECONFIG_CONTENT" | base64 -d > "$KUBECONFIG_PATH"
    - export KUBECONFIG="$KUBECONFIG_PATH" 
  script:
    - kubectl apply -f deployment-service.yml 
  tags:
    - self-hosted 
  only: 
      - main
```

**Explanation:**
- **Stage Name:** `deploy_to_kubernetes`, part of the `deploy` stage.
- **Purpose:** Applies Kubernetes manifest files to deploy the application.
- **Tags:** Runs on a runner with the `self-hosted` tag.

![Screenshot from 2024-08-11 19-22-01](https://github.com/user-attachments/assets/788b90cc-fa1b-4be3-bf05-d16e52169e09)


### Stage 10: Verify Deployment
Here, the successful deployment of the application is confirmed.

```yaml
verify_deployment:
  stage: deploy
  script:
    - kubectl get pods --namespace your_namespace
  tags:
    - kubernetes
```

**Explanation:**
- **Stage Name:** `verify_deployment`, part of the `deploy` stage.
- **Purpose:** Verifies deployment by listing pods in the namespace.
- **Tags:** Runs on a runner with the `self-hosted` tag.

![1720536087962](https://github.com/user-attachments/assets/509241e3-340f-41d6-bd4f-79dbaea058f9)


##
