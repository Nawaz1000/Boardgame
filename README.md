# BoardgameListingWebApp
A full-stack Java-based web application to manage and review board games with role-based access. Built using Spring
Boot and deployed using Azure DevOps pipelines and Kubernetes.
## Description
This application allows users to browse a list of board games and their reviews.
### Role Permissions:
- **Non-members**: Can view games and reviews.
- **Users**: Can add games and write reviews.
- **Managers**: Can edit and delete reviews.
## Technologies Used
- **Backend**: Java, Spring Boot, Spring MVC, Spring Security, JDBC
- **Frontend**: Thymeleaf, HTML5, CSS, JavaScript, Bootstrap
- **Database**: H2 (In-memory) with custom `schema.sql`
- **Testing**: JUnit
- **Build Tool**: Maven
- **CI/CD**: Azure DevOps (Classic Pipelines)
- **Security**: Trivy for vulnerability scanning
- **Code Quality**: SonarQube
- **Containerization**: Docker
- **Deployment**: Azure Kubernetes Service (AKS), optional AWS EC2

- ## Features
- Role-based access with Spring Security
- Full CRUD operations for games and reviews
- Authentication with username/password
- Reusable layouts with Thymeleaf Fragments
- CI/CD with automated scanning and analysis
- Kubernetes deployment with Ingress exposure
## How to Set Up & Deploy Using Azure DevOps
### Step 1: Import Repository
- Import this project into your Azure DevOps Repos.
### Step 2: Create a Build Pipeline (Classic Editor)
1. Go to **Pipelines > New Pipeline > Classic Editor**.
2. Select your repo.
3. Choose either:
 - Microsoft-hosted agent (simpler setup)
 - Self-hosted agent (for advanced control see next step)
### Step 3: Set Up a Self-hosted Agent (Optional)
1. Go to **Organization Settings > Pipelines > Agent Pools**.
2. Create a new agent pool.
3. In that pool, click **New Agent**, and follow install instructions based on OS.
4. On your VM:
 ```bash
 mkdir myagent && cd myagent
 wget https://vstsagentpackage.azureedge.net/agent/your-version.tar.gz
 tar zxvf your-version.tar.gz
 ./config.sh
 ./svc.sh install
 ./svc.sh start
 ```
5. Ensure the agent is online in Azure DevOps.
6. Add capabilities:
 - `JAVA=true`
 - `MAVEN=true`
### Step 4: Install Required Tools on Agent VM
```bash
sudo apt update
sudo apt install openjdk-17-jdk maven -y
sudo apt install docker.io -y
sudo usermod -aG docker $USER
wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.1_Linux-64bit.deb
sudo dpkg -i trivy_0.50.1_Linux-64bit.deb
```
### Step 5: Deploy AKS Kubernetes Cluster
1. Go to Azure Portal > **Kubernetes Services** > Create.
2. Settings:
 - Node count: min 1, max 30
 - VM size: D2s v3
 - Disable alerts
### Step 6: Create Release Pipeline
1. Go to **Pipelines > Releases > New Pipeline**.
2. Add an **Artifact** from the build output.
3. Create an **empty job** and configure tasks:
 ```bash
 kubectl apply -f deployment.yaml
 kubectl apply -f service.yaml
 ```
### Step 7: Configure CI Pipeline Tasks
1. Add **Maven task** to build:
 ```bash
 mvn clean install
 ```
2. Add **Trivy scan task**:
 ```bash
 trivy fs --format table -o trivy-report.html .
 ```
3. Add **Docker tasks**:
 ```bash
 docker build -t your-dockerhub-username/boardgame-app .
 docker push your-dockerhub-username/boardgame-app
 ```
### Step 8: Configure SonarQube
1. Run SonarQube on your VM:
 ```bash
 docker run -d --name sonarqube -p 9000:9000 sonarqube
 ```
2. Open `http://<VM-IP>:9000` and generate a token.
3. Add **SonarQube extension** in Azure DevOps and use token + server URL.
4. Add **Prepare and Analyze** tasks in your pipeline.
### Step 9: Access Your Deployed App
1. In Azure Portal:
 - Go to **AKS > Workloads** and confirm pod status.
 - Go to **AKS > Ingress** and copy the **external IP**.
2. Open your app in the browser using that IP.
## Run Unit Tests Locally
```bash
mvn clean test
```
## Docker (Local Testing)
```bash
docker build -t boardgame-app .
docker run -p 8080:8080 boardgame-app
```
## Project Structure
src/
 main/
 java/
 resources/
 webapp/
 test/
## Reference Video
[Watch CI/CD setup on YouTube](https://www.youtube.com/watch?v=0knqjEp3coU)
