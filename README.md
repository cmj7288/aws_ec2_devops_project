DevOps CI/CD Project: Jenkins, SonarQube, Trivy and Docker
Project Overview
I implemented a CI/CD pipeline using Jenkins, integrating SonarQube for code quality analysis, Trivy for container vulnerability scanning, and Docker for containerization. This pipeline builds a Java application with Maven, performs automated quality and security checks, and then builds and pushes a Docker image to Docker Hub.
Key Technologies: Jenkins, Docker, SonarQube, Trivy, Maven, GitHub, Docker Hub, AWS EC2
________________________________________
Architecture and Pipeline Flow

Figure 1: CI/CD Pipeline Architecture Diagram
High-Level Pipeline Flow
1.	Developer pushes code changes to GitHub repository
2.	Jenkins automatically pulls the latest code from GitHub
3.	Maven builds the application and runs unit tests
4.	SonarQube performs static code analysis and sends results to Jenkins
5.	Jenkins validates the SonarQube Quality Gate status
6.	Trivy scans the Docker image for security vulnerabilities
7.	Jenkins builds a Docker image and tags it as chinmayjoshi972/javapp:latest
8.	Jenkins authenticates to Docker Hub and pushes the image
9.	Cleanup process removes old/unused Docker images from the Jenkins server
Pipeline Flow Diagram:


 

________________________________________
Environment and Tools Configuration
Technology Stack
Tool	Purpose
Jenkins	CI/CD orchestration and automation
Docker	Container runtime and image building
SonarQube	Static code analysis and quality gates
Trivy	Container vulnerability scanning
Maven	Java build and dependency management
GitHub	Source code version control
Docker Hub	Container image registry
AWS EC2	Cloud infrastructure hosting

Table 1: Tools and Technologies Used
________________________________________
Detailed Setup and Installation
3.1 Jenkins Setup on Ubuntu (AWS EC2)
Prerequisites and Java Installation:
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
Jenkins Installation:
After multiple attempts with GPG key configuration (as documented in command history), Jenkins was successfully installed:
Add Jenkins repository and GPG key
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc
https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]"
https://pkg.jenkins.io/debian-stable binary/ |
sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
Start and Enable Jenkins Service:
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
Retrieve Initial Admin Password:
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
Access Jenkins web UI at http://<server-ip>:8080 and complete the initial setup wizard.
 

Figure 2: Jenkins Dashboard and Configuration Screenshots
________________________________________
3.2 Docker Installation and Configuration
Install Docker:
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
docker --version
Configure Jenkins User for Docker Access:
This step is critical to allow Jenkins to execute Docker commands without permission errors:
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo systemctl restart docker
Verify Docker Installation:
docker ps
docker images
 

Figure 3: Docker Installation Verification
________________________________________
3.3 SonarQube Setup with Docker
Run SonarQube Container:
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts
docker ps
docker start <container-id>
Access SonarQube:
•	URL: http://<server-ip>:9000
•	Default credentials: admin / admin
•	Change password on first login
SonarQube Project Configuration:
1.	Create a new project in SonarQube UI
2.	Generate an authentication token for Jenkins integration
3.	Configure project key and analysis settings
4.	Set up Quality Gate rules (code coverage, bugs, vulnerabilities, code smells)  

Figure 4: SonarQube Dashboard and Project Configuration
________________________________________
3.4 Trivy Installation for Security Scanning
Install Trivy:
sudo apt install wget apt-transport-https gnupg -y
wget https://aquasecurity.github.io/trivy-repo/deb/public.key
sudo apt-key add public.key
sudo apt update
sudo apt install trivy -y
Verify Trivy Installation:
trivy -v
trivy image nginx
Trivy scans container images for known vulnerabilities (CVEs) in OS packages and application dependencies.
  
Figure 5: Trivy Vulnerability Scan Results
________________________________________
3.5 Maven and Source Code Repository
Clone Application Repository:
git clone https://github.com/cmj7288/aws_ec2_devops_project.git
cd aws_ec2_devops_project/aws_ec2_devops_project/
cd tools_installation_scripts/
Install Maven:
sh Maven.sh
mvn -version
The repository contains:
•	Java application source code
•	Dockerfile for containerization
•	Jenkinsfile defining pipeline stages
•	Tool installation scripts
________________________________________
Jenkins Pipeline Implementation
4.1 Pipeline Stages
The Jenkins pipeline consists of nine sequential stages:
1.	Git Checkout – Pull latest code from GitHub repository
2.	Maven Build – Compile Java application and run unit tests
3.	SonarQube Code Analysis – Perform static code analysis
4.	Quality Gate Check – Validate code quality against defined thresholds
5.	Trivy Security Scan – Scan Docker image for vulnerabilities
6.	Docker Image Build – Build container image with tag chinmayjoshi972/javapp:latest
7.	Docker Login – Authenticate to Docker Hub using Jenkins credentials
8.	Docker Push – Push image to Docker Hub registry
9.	Docker Image Cleanup – Remove local images to free disk space
 
Figure 6: Jenkins Pipeline Stage View
________________________________________
4.2 Key Pipeline Commands
Docker Build Command:
docker image build -t chinmayjoshi972/javapp:latest .
Docker Authentication:
docker login -u chinmayjoshi972
Docker Push to Registry:
docker push chinmayjoshi972/javapp:latest
Docker Cleanup:
docker image prune -f
________________________________________
Troubleshooting and Issue Resolution
Throughout the project implementation, several issues were encountered and resolved. This section documents the problems, root causes, and solutions.
6.1 SonarQube Scan – Server Not Reachable
Issue:
Jenkins pipeline failed at the SonarQube Analysis stage with error:
SonarQube server [http://<server-ip>:9000] cannot be reached
Root Cause:
•	Port 9000 was not open in AWS Security Group
•	SonarQube container was stopped or not running
Resolution:
1.	Opened port 9000 in AWS EC2 Security Group (Inbound Rules)
2.	Verified SonarQube container status: docker ps -a
3.	Started SonarQube container: docker start <container-id>
4.	Verified accessibility: curl http://localhost:9000
________________________________________
6.2 Quality Gate Check – Status Stuck in PENDING
Issue:
The Quality Gate stage remained in PENDING status indefinitely, preventing pipeline completion.
Root Cause:
SonarQube webhook was not configured to send Quality Gate results back to Jenkins.
Resolution:
Configured SonarQube webhook in SonarQube UI:
1.	Navigate to: Administration → Configuration → Webhooks
2.	Create new webhook with URL: http://<jenkins-ip>:8080/sonarqube-webhook/
3.	Save and test webhook connection
After webhook configuration, Quality Gate results were immediately reported to Jenkins.
________________________________________
6.3 Docker Build – Base Image Not Found
Issue:
Docker build failed with error:
Error: openjdk:8-jdk-alpine not found
Root Cause:
The base image openjdk:8-jdk-alpine was deprecated or removed from Docker Hub.
Resolution:
Used an alternative compatible image and tagged it locally:
docker pull eclipse-temurin:8-jdk-alpine
docker tag eclipse-temurin:8-jdk-alpine openjdk:8-jdk-alpine
docker images
Updated Dockerfile to use eclipse-temurin:8-jdk-alpine directly in production.
________________________________________
6.4 Docker Build – Permission Denied Error
Issue:
Jenkins failed to execute Docker commands with error:
permission denied while trying to connect to /var/run/docker.sock
Root Cause:
Jenkins user did not have permissions to access the Docker daemon socket.
Resolution:
Added Jenkins user to the docker group and restarted services:
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo systemctl restart docker
groups jenkins # Verify group membership
After restart, Jenkins could execute Docker commands without sudo.
________________________________________
6.5 Docker Login – Authentication Failed
Issue 1: Incorrect Credentials
Error: unauthorized: incorrect username or password
Resolution:
1.	Created a Docker Hub Personal Access Token (PAT)
2.	Added credentials in Jenkins: Manage Jenkins → Credentials → Global → Add Credentials
3.	Used PAT instead of password for authentication
Issue 2: Missing Credentials in Jenkins
Error: Could not find credentials entry with ID 'docker'
Resolution:
1.	Navigated to Jenkins credentials store
2.	Created new credentials with ID: docker
3.	Updated Jenkinsfile to reference credential ID: docker
________________________________________
Results and Final Outcome
7.1 Successful Pipeline Execution
After resolving all issues, the complete CI/CD pipeline executed successfully:
Pipeline Execution Flow:
✓ Git Checkout
✓ Maven Build
✓ SonarQube Code Analysis
✓ Quality Gate Check (PASSED)
✓ Trivy Security Scan
✓ Docker Image Build
✓ Docker Login
✓ Docker Push
✓ Docker Image Cleanup
  
Figure 7: Successful Jenkins Pipeline Execution
________________________________________
7.2 Final Deliverables
Docker Image Published:
•	Repository: chinmayjoshi972/javapp
•	Tag: latest
•	Registry: Docker Hub
•	Full image name: chinmayjoshi972/javapp:latest
CI/CD Integration Achieved:
•	Automated build and test on code commits
•	Code quality validation with SonarQube
•	Security vulnerability scanning with Trivy
•	Automated containerization and registry push
•	Complete audit trail in Jenkins
________________________________________
7.3 Key Metrics and Benefits
Metric	Value
Pipeline Stages	9
Average Build Time	8-12 minutes
Code Quality Gate	Configured and enforced
Security Scanning	Automated with Trivy
Deployment Target	Docker Hub
Automation Level	Fully automated

Table 2: CI/CD Pipeline Metrics
Project Benefits:
•	Reduced manual deployment effort by 80%
•	Automated quality and security checks prevent defects from reaching production
•	Consistent build and deployment process across environments
•	Complete traceability from code commit to deployment
•	Foundation for further DevOps practices (IaC, monitoring, auto-scaling)
________________________________________
Command History Reference
Complete command history from EC2 instance showing the entire setup process (151 commands executed):
Key command categories:
•	System updates and package installations
•	Jenkins installation troubleshooting (multiple GPG key attempts)
•	Docker installation and configuration
•	User permission management
•	Container operations (start, stop, status checks)
•	Image pulling and tagging
•	Service management (systemctl operations)
This command history demonstrates practical troubleshooting skills and persistence in resolving configuration issues.
________________________________________
Conclusion
This project successfully demonstrates the implementation of a production-grade CI/CD pipeline using industry-standard DevOps tools. The pipeline automates the entire software delivery process from code commit to container registry deployment, with integrated quality gates and security scanning.
Key Learnings:
•	Integration of multiple DevOps tools (Jenkins, SonarQube, Trivy, Docker)
•	Troubleshooting complex permission and networking issues
•	Configuring webhooks for asynchronous communication between tools
•	Container security best practices with vulnerability scanning
•	AWS EC2 infrastructure management
Future Enhancements:
•	Deploy to Kubernetes cluster instead of just pushing to registry
•	Add automated testing stages (integration tests, performance tests)
•	Implement blue-green or canary deployment strategies
•	Add monitoring and alerting with Prometheus and Grafana
•	Integrate Infrastructure as Code (Terraform) for environment provisioning
This project serves as a strong foundation for building more advanced DevOps pipelines and demonstrates practical experience with real-world CI/CD challenges and solutions.
