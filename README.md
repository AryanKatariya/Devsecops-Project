# 🚀 DevSecOps Pipeline Project

![Build Status](https://img.shields.io/badge/build-passing-brightgreen) 

This project integrates comprehensive security checks into a Jenkins pipeline to automate and secure the software development lifecycle. The pipeline includes secret scanning, software composition analysis, static code analysis, dynamic application testing, and deployment, ensuring that security is integrated at every stage of the CI/CD process.

## 📋 Table of Contents
- [Overview](#overview)
- [Pipeline Stages](#pipeline-stages)
- [Technologies Used](#technologies-used)
- [Prerequisites](#prerequisites)
- [Environment Variables](#environment-variables)
- [Installation](#installation)
- [Usage](#usage)
- [Configurations](#configurations)
- [Contributors](#contributors)

## 📝 Overview

This DevSecOps pipeline ensures security is integrated at every stage of development. Key features include:
- **Secret Scanning** : Detects sensitive information (e.g., API keys) using TruffleHog.
- **Software Composition Analysis(SCA)** : Identifies vulnerabilities in dependencies using OWASP Dependency Check.
- **Static Application Security Testing(SAST)**: Analyzes the codebase for vulnerabilities using SonarQube.
- **Dynamic Application Security Testing(DAST)** :Tests the deployed application for vulnerabilities using OWASP ZAP.

## 🛠 Pipeline Stages

| **Stage**             | **Tool**               | **Description**                                             |
| --------------------- | ---------------------- | ----------------------------------------------------------- |
| **Cleanup Workspace** | Jenkins (cleanWs)      | Cleans up the workspace before starting                     |
| **Checkout from SCM** | Git                    | Clones the code from GitHub                                 |
| **Secret Scanning**   | TruffleHog             | Scans for sensitive information like API keys and passwords |
| **SCA**               | OWASP Dependency Check | Identifies vulnerabilities in dependencies                  |
| **Start SonarQube**   | Docker (SonarQube)     | Spins up a SonarQube instance for analysis                  |
| **SAST**              | SonarQube              | Analyzes source code for security issues                    |
| **Build Application** | Maven                  | Builds the application, skipping tests                      |
| **Deploy to Server**  | SCP (via SSH)          | Deploys the application to the target server                |
| **DAST**              | OWASP ZAP              | Performs dynamic security tests on the running application  |

## 🚀 Technologies Used

- **Jenkins**: For orchestrating the CI/CD pipeline.
- **TruffleHog**: For secret scanning.
- **OWASP Dependency Check**: For Software Composition Analysis.
- **SonarQube**: For Static Application Security Testing (SAST).
- **Maven**: For building and managing dependencies.
- **OWASP ZAP**: For Dynamic Application Security Testing (DAST).
- **Docker**: For containerizing SonarQube and OWASP ZAP.
- **DefectDojo**: For vulnerability management and importing scan results.

## 📋 Prerequisites

Ensure you have the following installed on your local machine or server:

- **Jenkins**
- **Docker**
- **Maven**
- **SonarQube**
- **OWASP Dependency Check**
- **TruffleHog**
- **OWASP ZAP**

### 🔑 Environment Variables

Set up the following environment variables in Jenkins:

- `GITHUB_TOKEN`: GitHub Personal Access Token
- `API_KEY`: DefectDojo API Key
- `DOJO_IP`: DefectDojo Server IP
- `deploy-ssh`: SSH credentials ID for remote deployment

## 🔧 Installation

1. **Clone the repository**:
    ```bash
    git clone https://github.com/AryanKatariya/Devsecops-Project.git
    cd Devsecops-Project
    ```

2. **Setup Jenkins**:
   - Install necessary Jenkins plugins: Git, Maven Integration, SonarQube Scanner, SSH Agent, etc.
   - Configure Jenkins credentials for GitHub and SSH deployment.

3. **Run SonarQube in Docker**:
    ```bash
    docker run -d --name sonar -p 9000:9000 -v sonarqube_data:/opt/sonarqube/data sonarqube:lts-community
    ```

4. **Configure OWASP ZAP**: Install OWASP ZAP and ensure it is available on the deployment target for DAST.

## 🚀 Usage

To use this pipeline:

1. Push your code to the GitHub repository.
2. The Jenkins pipeline will be automatically triggered, running through all the security checks and deploying the application.

You can also manually trigger the Jenkins pipeline if required.

## ⚙️ Configuration

- Modify the `Jenkinsfile` to suit your deployment environment, including:
  - GitHub repository URL
  - Target server IP for deployment
  - DefectDojo IP and API key
  - Specifics for the scan type and scan parameters
- Adjust `sonar-project.properties` for custom SonarQube settings.

## 🤝 Contributors
| ![Aryan](https://github.com/AryanKatariya.png?size=50) | ![Chaitanya](https://github.com/chaitanyaodd1.png?size=50) | ![Gaurav](https://github.com/greninja05.png?size=50) | ![Ayush](https://github.com/ayushpujara.png?size=0) |
| :----------------------------------------------------: | :--------------------------------------------------------: | :--------------------------------------------------: | --------------------------------------------------- |
|                   **Aryan Katariya**                   |                   **Chaitanya Dankhade**                   |                  **Gaurav Meshram**                  | **Ayush Pujara**                                    |
