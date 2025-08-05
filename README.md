# 🚀 Task 2 - CI/CD Pipeline with Jenkins, SonarQube, DockerHub

A complete CI/CD pipeline for a Java-based web application (`student2-ui`) using **Jenkins**, **SonarQube**, **GitLeaks**, **Trivy**, and **Docker**, deployed on an **AWS EC2 instance**.

---

### 📦 Installation & Setup

1.  **Clone the Repository**

    ```bash
    git clone [https://github.com/chetan2624/student2-ui.git](https://github.com/chetan2624/student2-ui.git)
    cd student2-ui
    ```

2.  **Launch AWS EC2 Instance & Install Dependencies**

    Launch an EC2 instance with Ubuntu and install the necessary dependencies:

    ```bash
    sudo apt update
    sudo apt install -y openjdk-11-jdk maven git docker.io
    ```

3.  **Install Jenkins**

    Install Jenkins on your EC2 instance:

    ```bash
    wget -q -O - [https://pkg.jenkins.io/debian/jenkins.io.key](https://pkg.jenkins.io/debian/jenkins.io.key) | sudo apt-key add -
    sudo sh -c 'echo deb [http://pkg.jenkins.io/debian-stable](http://pkg.jenkins.io/debian-stable) binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt update
    sudo apt install -y jenkins
    sudo systemctl start jenkins
    ```

    Access Jenkins via your browser: `http://<your-ec2-ip>:8080`

4.  **Install GitLeaks**

    Install GitLeaks to scan for secrets in your code:

    ```bash
    curl -sSL [https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks-linux-amd64](https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks-linux-amd64) -o gitleaks
    chmod +x gitleaks
    sudo mv gitleaks /usr/local/bin/
    ```

5.  **Install Trivy**

    Install Trivy to scan Docker images for vulnerabilities:

    ```bash
    sudo apt install -y wget apt-transport-https gnupg lsb-release
    wget -qO - [https://aquasecurity.github.io/trivy-repo/deb/public.key](https://aquasecurity.github.io/trivy-repo/deb/public.key) | sudo apt-key add -
    echo "deb [https://aquasecurity.github.io/trivy-repo/deb](https://aquasecurity.github.io/trivy-repo/deb) $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
    sudo apt update
    sudo apt install -y trivy
    ```

6.  **Set Up SonarQube (Using Docker)**

    Run SonarQube as a Docker container:

    ```bash
    docker run -d --name sonarqube -p 9000:9000 sonarqube:lts
    ```

    Access it at: `http://<your-ec2-ip>:9000`

7.  **Jenkins Configuration**

    Install the required Jenkins plugins:
    -   Git
    -   Pipeline
    -   Docker Pipeline
    -   SonarQube Scanner

    Configure SonarQube in Jenkins:
    -   Go to **Manage Jenkins > Configure System**
    -   Add SonarQube server with the name: `Sonarqube`

    Add DockerHub credentials in Jenkins:
    -   Go to **Manage Jenkins > Manage Credentials > Global**
    -   Add new credentials with:
        -   ID: `docker-hub-credentials`
        -   Type: `Username with Password`
        -   Username: your DockerHub username
        -   Password: your DockerHub password or token

---

### 🧩 Jenkinsfile Pipeline Breakdown

Below are the stages defined in the `Jenkinsfile` for automating the CI/CD process.

-   **Stage 1: Clone the Repository**
    ```groovy
    git '[https://github.com/chetan2624/student2-ui.git](https://github.com/chetan2624/student2-ui.git)'
    ```

-   **Stage 2: GitLeaks - Scan for Secrets**
    ```bash
    gitleaks detect --source . --report=gitleaks-report.json || true
    ```

-   **Stage 3: Build the App using Maven**
    ```bash
    mvn clean package
    ```

-   **Stage 4: SonarQube Analysis**
    ```bash
    mvn clean verify sonar:sonar \
      -Dsonar.projectKey=student2-ui \
      -Dsonar.projectName="student2-ui"
    ```

-   **Stage 5: Build Docker Image**
    ```bash
    docker build -t chetan801/student2-ui:latest .
    ```

-   **Stage 6: Scan Docker Image using Trivy**
    ```bash
    trivy image chetan801/student2-ui:latest || true
    ```

-   **Stage 7: Push Docker Image to Docker Hub**
    ```bash
    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
    docker push chetan801/student2-ui:latest
    ```
    *These credentials are securely pulled from Jenkins using the `docker-hub-credentials` ID.*

-   **Stage 8: Clean Workspace and Docker Resources**
    ```bash
    docker system prune -af || true
    cleanWs()
    ```

---

### ✅ Outcome

By the end of this task, we achieved:
-   🔁 Fully automated CI/CD using Jenkins
-   🔍 Secret scanning with GitLeaks
-   ✅ Code quality checks via SonarQube
-   🐳 Docker image build and vulnerability scan
-   🚀 Docker image deployment to Docker Hub
-   🧹 Workspace and Docker cleanup after the pipeline run

### 📁 Repository
-   GitHub Repository: `https://github.com/chetan2624/student2-ui`

### 📬 Contact
For issues or questions, feel free to open an issue in the repo or reach out to the project maintainer.

