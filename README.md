<div align="center">
  <img src="./public/assets/First.png" alt="Logo" width="100%" height="100%">

  <br>
    <img src="./public/assets/netflix-logo.png" alt="Logo" width="100" height="32">
  </a>
</div>

<br />

<div align="center">
  <img src="./public/assets/Second.png" alt="Logo" width="100%" height="100%">
  <p align="center">Home Page</p>
</div>

# Deploy Movies Website Clone on Cloud Using Jenkins-Project

### **Phase 1: Initial Setup and Deployment**

**Step 1: Launch EC2 Instance**

- Provision an EC2 instance on AWS running Ubuntu 22.04.
- Connect to the instance using SSH.

**Step 2: Clone the Code:**

- Update the packages on the EC2 instance.
- ```bash
  git clone https://github.com/AmrTarek77/Deploy_Movies_Website_Clone_on_Cloud_Using_Jenkins-Project.git
  ```

**Step 3: Install Docker and Run the App Using a Container:**

- Set up Docker on the EC2 instance:

  ```bash

  sudo apt-get update
  sudo apt-get install docker.io -y
  sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
  newgrp docker
  sudo chmod 777 /var/run/docker.sock
  ```

- Build and run the Movies Clone application using Docker containers:

  ```bash
  docker build -t movies .
  docker run -d --name movies -p 8081:80 movies:latest

  #to delete
  docker stop <containerid>
  docker rmi -f movies
  ```

Note: An error will occur because an API key is required.

**Step 4: Get the API Key:**

- Open a web browser and go to the TMDB (The Movie Database) website.
- Click on "Login" and create a new account.
- After logging in, go to your profile and select "Settings."
- Click on "API" from the left-side panel.
- Create a new API key by clicking "Create" and accepting the terms and conditions.
- Provide the necessary details and click "Submit."
- You will receive your TMDB API key.

Now recreate the Docker image with your api key:

```bash
docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t movies .
docker run -d --name movies -p 8081:80 movies:latest
```

**Phase 2: Security**

1. **Install SonarQube and Trivy:**

   - Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.

     ```bash
     docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
     ```

     To access:

     - Open a web browser and enter the EC2 instance's public IP followed by port 9000 (e.g., http://public_ip:9000).
     - Use the default username and password, which is 'admin'.

     To install Trivy:

     ```bash
     sudo apt-get install wget apt-transport-https gnupg lsb-release
     wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
     echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
     sudo apt-get update
     sudo apt-get install trivy
     ```

     to scan image using trivy

     ```bash
     trivy image <image_name>
     ```

2. **Integrate SonarQube and Configure:**
   - Integrate SonarQube with your CI/CD pipeline.
   - Configure SonarQube to analyze code for quality and security issues.

**Phase 3: CI/CD Setup**

1. **Install Jenkins for Automation:**

   - Install Jenkins on the EC2 instance to automate deployment:
   - Install Java and Jenkins:

   ```bash
   sudo apt update
   sudo apt install fontconfig openjdk-17-jre
   java -version
   openjdk version "17.0.8" 2023-07-18
   OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
   OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)

   # Jenkins installation
   sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
   https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
   https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
   /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

   - Access Jenkins using a web browser and the EC2 instance's public IP followed by port 8080 (e.g., http://public_ip:8080).

2. **Install Necessary Plugins in Jenkins:**

- In the Jenkins web interface, go to "Manage Jenkins" -> "Manage Plugins" -> "Available" tab.

- Install the following plugins:

  1.  Eclipse Temurin Installer (Install without restart)
  2.  SonarQube Scanner (Install without restart)
  3.  NodeJs Plugin (Install without restart)
  4.  Email Extension Plugin

### **Configure Java and Nodejs in Global Tool Configuration**

- In the Jenkins web interface, go to "Manage Jenkins" -> "Global Tool Configuration" -> "JDK" section.
- Install JDK 17 and Node.js 16 by providing their respective installation paths.
- Click on "Apply" and "Save" to save the configuration.

### SonarQube Configuration

- Create a token for SonarQube:

  - Go to the Jenkins Dashboard -> "Manage Jenkins" -> "Credentials" -> "Add Secret Text".
  - Add the SonarQube token and save it.

- Configure SonarQube in Jenkins:
  - Go to the Jenkins Dashboard -> "Manage Jenkins" -> "Configure System".
  - Set up the SonarQube installation and provide the SonarQube server URL and token.

**The Configure System option** is used in Jenkins to configure different server

**Global Tool Configuration** is used to configure different tools that are installed using Plugins. For example, we will install a Sonar scanner in the tools.

We will install a sonar scanner in the tools.

To create a Jenkins webhook, follow these steps:

1. **Configure CI/CD Pipeline in Jenkins:**

```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
}
```

Here are the instructions without step numbers:

**Install Dependency-Check and Docker Tools in Jenkins**

**Install Dependency-Check Plugin:**

1. Go to "Dashboard" in your Jenkins web interface.
2. Navigate to "Manage Jenkins" → "Manage Plugins."
3. Click on the "Available" tab and search for "OWASP Dependency-Check."
4. Check the checkbox for "OWASP Dependency-Check" and click on the "Install without restart" button.

**Configure Dependency-Check Tool:**

- After installing the Dependency-Check plugin, you need to configure the tool.
- Go to "Dashboard" → "Manage Jenkins" → "Global Tool Configuration."
- Find the section for "OWASP Dependency-Check."
- Add the tool's name, e.g., "DP-Check."
- Save your settings.

**Install Docker Tools and Docker Plugins:**

- Go to "Dashboard" in your Jenkins web interface.
- Navigate to "Manage Jenkins" → "Manage Plugins."
- Click on the "Available" tab and search for "Docker."
- Check the following Docker-related plugins:
  - Docker
  - Docker Commons
  - Docker Pipeline
  - Docker API
  - docker-build-step
- Click on the "Install without restart" button to install these plugins.

**Add DockerHub Credentials:**

- To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:
  - Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."
  - Click on "System" and then "Global credentials (unrestricted)."
  - Click on "Add Credentials" on the left side.
  - Choose "Secret text" as the kind of credentials.
  - Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker").
  - Click "OK" to save your DockerHub credentials.

Now, you have installed the Dependency-Check plugin, configured the tool, and added Docker-related plugins along with your DockerHub credentials in Jenkins. You can now proceed with configuring your Jenkins pipeline to include these tools and credentials in your CI/CD process.

```

pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build --build-arg TMDB_V3_API_KEY=<yourapikey> -t netflix ."
                       sh "docker tag netflix nasi101/netflix:latest "
                       sh "docker push nasi101/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image nasi101/netflix:latest > trivyimage.txt"
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 nasi101/netflix:latest'
            }
        }
    }
}
```

If you encounter a "docker login failed" error, you can try the following alternative steps:

1.  Switch to the root user:

```bash

sudo su
```

2.  Add the "jenkins" user to the "docker" group:

```bash

sudo usermod -aG docker jenkins
```

3.  Restart the Jenkins service:

```bash

sudo systemctl restart jenkins
```

These steps will give the "jenkins" user the necessary permissions to access the Docker daemon and resolve the "docker login failed" error.

**Phase 4: Monitoring**

1. **Install Prometheus and Grafana:**

   Set up Prometheus and Grafana to monitor your application.

   **Installing Prometheus:**

   First, create a dedicated Linux user for Prometheus and download Prometheus:

   ```bash
   sudo useradd --system --no-create-home --shell /bin/false prometheus
   wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
   ```


Extract Prometheus files, move them, and create directories:

```bash
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

Set ownership for directories:

```bash
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

Create a systemd unit configuration file for Prometheus:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Add the following content to the `prometheus.service` file:

```plaintext
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

Here's a brief explanation of the key parts in this `prometheus.service` file:

- `User` and `Group` specify the Linux user and group under which Prometheus will run.

- `ExecStart` is where you specify the Prometheus binary path, the location of the configuration file (`prometheus.yml`), the storage directory, and other settings.

- `web.listen-address` configures Prometheus to listen on all network interfaces on port 9090.

- `web.enable-lifecycle` allows for management of Prometheus through API calls.

Enable and start Prometheus:

```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

Verify Prometheus's status:

```bash
sudo systemctl status prometheus
```

You can access Prometheus in a web browser using your server's IP and port 9090:

`http://<your-server-ip>:9090`

**Installing Node Exporter:**

Create a system user for Node Exporter and download Node Exporter:

```bash
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```

Extract Node Exporter files, move the binary, and clean up:

```bash
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
```

Create a systemd unit configuration file for Node Exporter:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Add the following content to the `node_exporter.service` file:

```plaintext
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target
```

Replace `--collector.logind` with any additional flags as needed.

Enable and start Node Exporter:

```bash
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

Verify the Node Exporter's status:

```bash
sudo systemctl status node_exporter
```

You can access Node Exporter metrics in Prometheus.

2. **Configure Prometheus Plugin Integration:**

   Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

   **Prometheus Configuration:**

   To configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the `prometheus.yml` file. Here is an example `prometheus.yml` configuration for your setup:

   ```yaml
   global:
     scrape_interval: 15s

   scrape_configs:
     - job_name: "node_exporter"
       static_configs:
         - targets: ["localhost:9100"]

     - job_name: "jenkins"
       metrics_path: "/prometheus"
       static_configs:
         - targets: ["<your-jenkins-ip>:<your-jenkins-port>"]
   ```

   Make sure to replace `<your-jenkins-ip>` and `<your-jenkins-port>` with the appropriate values for your Jenkins setup.

   Check the validity of the configuration file:

   ```bash
   promtool check config /etc/prometheus/prometheus.yml
   ```

   Reload the Prometheus configuration without restarting:

   ```bash
   curl -X POST http://localhost:9090/-/reload
   ```

   You can access Prometheus targets at:

   `http://<your-prometheus-ip>:9090/targets`

####Grafana

**Installing Grafana on Ubuntu 22.04 and Configuring it to Work with Prometheus**

**Step 1: Install Dependencies:**

First, ensure that all necessary dependencies are installed:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
```

**Step 2: Add the GPG Key:**

Add the GPG key for Grafana:

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

**Step 3: Add Grafana Repository:**

Add the repository for Grafana stable releases:

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

**Step 4: Update and Install Grafana:**

Update the package list and install Grafana:

```bash
sudo apt-get update
sudo apt-get -y install grafana
```

**Step 5: Enable and Start Grafana Service:**

To automatically start Grafana after a reboot, enable the service:

```bash
sudo systemctl enable grafana-server
```

Then, start Grafana:

```bash
sudo systemctl start grafana-server
```

**Step 6: Check Grafana Status:**

Verify the status of the Grafana service to ensure it's running correctly:

```bash
sudo systemctl status grafana-server
```

**Step 7: Access Grafana Web Interface:**

Open a web browser and navigate to Grafana using your server's IP address. The default port for Grafana is 3000. For example:

`http://<your-server-ip>:3000`

You'll be prompted to log in to Grafana. The default username is "admin," and the default password is also "admin."

**Step 8: Change the Default Password:**

When you log in for the first time, Grafana will prompt you to change the default password for security reasons. Follow the prompts to set a new password.

**Step 9: Add Prometheus Data Source:**

To visualize metrics, you need to add a data source. Follow these steps:

- Click on the gear icon (⚙️) in the left sidebar to open the "Configuration" menu.

- Select "Data Sources."

- Click on the "Add data source" button.

- Choose "Prometheus" as the data source type.

- In the "HTTP" section:
  - Set the "URL" to `http://localhost:9090` (assuming Prometheus is running on the same server).
  - Click the "Save & Test" button to ensure the data source is working.

**Step 10: Import a Dashboard:**

To make it easier to view metrics, you can import a pre-configured dashboard. Follow these steps:

- Click on the "+" (plus) icon in the left sidebar to open the "Create" menu.

- Select "Dashboard."

- Click on the "Import" dashboard option.

- Enter the dashboard code you want to import (e.g., code 1860).

- Click the "Load" button.

- Select the data source you added (Prometheus) from the dropdown.

- Click on the "Import" button.

You should now have a Grafana dashboard set up to visualize metrics from Prometheus.

Grafana is a powerful tool for creating visualizations and dashboards, and you can further customize it to suit your specific monitoring needs.

That's it! You've successfully installed and set up Grafana to work with Prometheus for monitoring and visualization.

2. **Configure Prometheus Plugin Integration:**
   - Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

**Phase 5: Notification**

1. **Implement Notification Services:**
   - Configure email notifications in Jenkins or any other preferred notification   mechanism.

# Phase 6: Kubernetes

## Create a Kubernetes Cluster with Node Groups

During this phase, you will establish a Kubernetes cluster with node groups. This setup will provide a scalable environment for deploying and managing your applications.

## Monitor Kubernetes with Prometheus

Prometheus is a robust monitoring and alerting toolkit that will be utilized to monitor your Kubernetes cluster. Additionally, you will employ Helm to install the node exporter, enabling the collection of metrics from your cluster nodes.

### Install Node Exporter via Helm

To initiate the monitoring of your Kubernetes cluster, follow these steps to install the Prometheus Node Exporter using Helm:

1. Add the Prometheus Community Helm repository:

   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   ```

2. Create a Kubernetes namespace for the Node Exporter:

   ```bash
   kubectl create namespace prometheus-node-exporter
   ```

3. Install the Node Exporter using Helm:

   ```bash
   helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter
   ```

Add a Job to Scrape Metrics on nodeip:9001/metrics in prometheus.yml:

Update your Prometheus configuration (prometheus.yml) to include a new job for scraping metrics from nodeip:9001/metrics. Add the following configuration to your prometheus.yml file:

```
  - job_name: 'Netflix'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['node1Ip:9100']
```

Replace 'your-job-name' with a descriptive name for your job. The static_configs section specifies the targets to scrape metrics from, and in this case, it's set to nodeip:9001.

Remember to reload or restart Prometheus to apply these configuration changes.

To deploy an application with ArgoCD, refer to the following steps provided in Markdown format:

### Deploy Application with ArgoCD

1. **Install ArgoCD:**

   You can install ArgoCD on your Kubernetes cluster by following the instructions provided in the [EKS Workshop](https://archive.eksworkshop.com/intermediate/290_argocd/install/) documentation.

2. **Set Your GitHub Repository as a Source:**

   After installing ArgoCD, configure your GitHub repository as a source for deploying your application. This typically involves establishing the connection to your repository and defining the source for your ArgoCD application. The specific steps will depend on your setup and requirements.

3. **Create an ArgoCD Application:**

   - `name`: Set the name for your application.
   - `destination`: Define the destination where your application should be deployed.
   - `project`: Specify the project the application belongs to.
   - `source`: Set the source of your application, including the GitHub repository URL, revision, and the path to the application within the repository.
   - `syncPolicy`: Configure the sync policy, including automatic syncing, pruning, and self-healing.

4. **Access your Application**
   - To Access the app make sure port 30007 is open in your security group and then open a new tab paste your NodeIP:30007, your app should be running.

**Phase 7: Cleanup**

1. **Cleanup AWS EC2 Instances:**
   - Terminate AWS EC2 instances that are no longer needed.


