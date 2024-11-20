In this Project, I will walk you through the process of setting up a **CI/CD pipeline** for a **Spring Boot banking application**. This pipeline will automate the build, deployment, and integration processes using **Jenkins**, **Docker**, and **GitHub**. You will learn how to leverage a **multi-node Jenkins architecture**, **shared libraries**, and **webhooks** for automatic deployments.

**Repository for this Project**: Used this Repo + Shared library repo : https://github.com/Amitabh-DevOps/Jenkins-shared-libraries

**Check Blog** : https://amitabhdevops.hashnode.dev/spring-boot-bank-jenkins 

> Use the `dev` branch for the code related to this project.

---

## **Project Overview**

This project involves creating a complete CI/CD pipeline that automates the deployment of a Spring Boot-based banking application. Here are the steps we will follow:

1. **Create AWS EC2 Instances** to host Jenkins and Docker.
    
2. **Set up Jenkins** to automate the CI/CD pipeline.
    
3. **Containerize the Spring Boot application** using Docker.
    
4. **Integrate GitHub** for automatic deployment triggered by code changes.
    
5. **Use a multi-node Jenkins setup** to deploy the application on a development server.
    
6. Create a **Jenkinsfile** for automated builds and deployment.
    
7. **Set up webhooks** in GitHub to trigger Jenkins builds on code changes.
    

---

## Steps to Implement the Project

To set up a **Jenkins Master-Agent** architecture on AWS, we will create two EC2 instances. The Jenkins Master instance will manage Jenkins, while the Jenkins Agent instance, with higher resources, will host and deploy the Spring bootapplication.

### Step 1: Create Two AWS EC2 Instances

We'll start by setting up two separate instances: one for the **Jenkins Master** and one for the **Jenkins Agent**.

1. **Log in to AWS**:  
    Go to the [AWS Console](https://aws.amazon.com/console/) and log in.
    
2. **Launch an EC2 Instance (Jenkins Master)**:
    
    * Go to the **EC2 Dashboard** and click on **Launch Instance**.
        
    * Select the **Ubuntu 24.04 LTS** AMI.
        
    * Choose **t2.micro** for the Jenkins Master instance, eligible for the free tier.
        
    * Configure **Security Group**:
        
        * **SSH (port 22)** for remote access.
            
        * **HTTP (port 80)** to access Jenkins through the browser.
            
    * Click **Review and Launch**.
        
3. **Launch an EC2 Instance (Jenkins Agent)**:
    
    * Repeat the above steps, but select **t2.medium** for the Jenkins Agent instance.
        
    * Use the **same key pair** as used for the Jenkins Master.
        

> Note: For simplicity and consistency, it’s best to use the same key pair for both instances. This enables both instances to be accessed with a single private key file (e.g., `<your-key>.pem`), which is useful for managing both servers in a DevOps environment.

### Step 2: Connect to Each EC2 Instance

SSH into both instances using:

```bash
ssh -i <your-key>.pem ubuntu@<your-ec2-public-ip>
```

### Step 3: Update Each EC2 Instance

Ensure both instances are up-to-date by running:

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 4: Install Java on Both Instances

Jenkins requires Java, so install OpenJDK 17 on each instance:

```bash
sudo apt install openjdk-17-jre -y
java -version
```

### Step 5: Install Jenkins (Only on Jenkins Master)

1. **Install dependencies**:
    
    ```bash
    sudo apt-get install -y ca-certificates curl gnupg
    ```
    
2. **Add the Jenkins repository key**:
    
    ```bash
    curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    ```
    
3. **Add Jenkins to APT sources**:
    
    ```bash
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
    ```
    
4. **Install Jenkins**:
    
    ```bash
    sudo apt-get update
    sudo apt-get install jenkins -y
    ```
    
5. **Enable Jenkins to start on boot**:
    
    ```bash
    sudo systemctl enable jenkins
    sudo systemctl start jenkins
    ```
    
6. **Verify Jenkins status**:
    
    ```bash
    sudo systemctl status jenkins
    ```
    

### Step 6: Install Docker on Both Instances

1. **Install Docker**:
    
    ```bash
    sudo apt install docker.io -y
    ```
    
2. **Add Jenkins to the Docker group** (on both instances):
    
    ```bash
    sudo usermod -aG docker $USER
    ```
    
3. **Refresh Docker group membership**:
    
    ```bash
    newgrp docker
    ```
    

### Step 7: Install Docker Compose on Both Instances

```bash
sudo apt install docker-compose-v2 -y
docker --version
docker-compose --version
```

### Step 8: Configure Jenkins Security Groups for Web Access

Edit the security group of your Jenkins Master instance:

1. Go to the **EC2 Dashboard**, select **Security Groups**, and choose the security group associated with your EC2 instance.
    
2. Click on **Edit Inbound Rules** and add a rule for **Custom TCP Rule** with **port 8080**.
    
3. Access Jenkins in a web browser using `http://<your-ec2-public-ip>:8080`.
    

### Step 9: Retrieve Jenkins Admin Password

Retrieve the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Use this password to complete the initial setup in Jenkins by following the on-screen instructions.

---

## Creating a Development Server from Jenkins Agent Instance to Deploy Spring boot bank App

To add a new node in Jenkins:

1. **Log in to Jenkins**.
    
2. Go to **Manage Jenkins &gt; Manage Nodes and Clouds**.
    
3. Click **New Node**:
    
    * **Node name**: Enter a name for the Jenkins Agent (e.g., `dev-server`).
        
    * Choose **Permanent Agent** and click **OK**.
        
4. **Configure Node Settings**:
    
    * **Remote root directory**: `/home/ubuntu/bankapp`
        
    * **Labels**: Add `dev-server`
        
    * **Usage**: Choose **Only build jobs with label expressions matching this node**.
        
5. Under **Launch method**, select **Launch agents via SSH**:
    
    * **Host**: Enter the private IP of your Jenkins Agent instance.
        
    * **Credentials**: Add credentials by selecting **SSH Username with private key**.
        
        * Use **ubuntu** for the username.
            
        * Add the private key associated with the key pair used for the Jenkins Agent EC2 instance.
            
    * Click **Save** and connect to the Jenkins Agent.
        

---

### Step 10: Set Up Docker Hub Credentials in Jenkins

1. Go to **Manage Jenkins &gt; Security &gt; Credentials &gt; System &gt; Global credentials (unrestricted)** and click **Add Credentials**.
    
2. Set **Kind** to **Username with password**.
    
3. Add your Docker Hub username and password and save , for password generate Personal Access Token on DockerHub.
    

---

### Step 11: Create a Jenkins Pipeline Job

1. **Create a New Job**:
    
    * From the Jenkins dashboard, click on **New Item**.
        
    * Enter a name (e.g., `Springboot bank CI/CD`), select **Pipeline**, and click **OK**.
        
2. **Configure GitHub Integration**:
    
    * In the **General** section, check the **GitHub project** option.
        
    * Provide the URL of your GitHub repository.
        
3. **Pipeline**:
    
    * Under **Pipeline**, select **Pipeline script from SCM**.
        
    * Set **SCM** to **Git** and provide the Git repository URL.
        
    * Choose the `dev` branch and set **Script Path** to `Jenkinsfile`.
        

---

### Step 12: Create the Jenkinsfile on GitHub

In the GitHub repository, create a `Jenkinsfile` containing the pipeline script:

```yaml
@Library('Shared')_

pipeline{
    agent {label 'dev-server'}
    
    stages{
        stage("Code"){
            steps{
                clone("https://github.com/Amitabh-DevOps/banking-app-project.git","dev")
                echo "Code clonning done."
            }
        }
        stage("Build"){                                                             
            steps{
                dockerbuild("bankapp-mini","latest")
                echo "Code build bhi hogaya."
            }
        }
        stage("Push to DockerHub"){
            steps{
                dockerpush("dockerHub","bankapp-mini","latest")
                echo "Push to dockerHub is also done."
            }
        }
        stage("Deplying"){
            steps{
                deploy()
                echo "Deployment bhi done."
            }
        }
    }
}
```

This script pulls the code from GitHub, builds and pushes a Docker image to Docker Hub, and deploys it on the Jenkins Agent instance.

* This script includes multiple stages: cloning the code from GitHub, building the Docker image, pushing it to Docker Hub, and deploying the container.
    
* This script allows to used shared library repo which is stored on my github profile
    
* **<mark>used shared library repo</mark>** <mark> : </mark> [<mark>https://github.com/Amitabh-DevOps/Jenkins-shared-libraries</mark>](https://github.com/Amitabh-DevOps/Jenkins-shared-libraries)
    
* **Commit the Changes**:
    
    * Save and commit the `Jenkinsfile` to your GitHub repository.
        

---

### Step 13. Set Up Webhooks for Automatic Deployment

To trigger the Jenkins pipeline automatically on code changes, set up webhooks in your GitHub repository.

1. **Go to GitHub Repository Settings**:
    
    * Navigate to your GitHub repository, and click on **Settings**.
        
2. **Set Up Webhooks**:
    
    * In the left sidebar, click on **Webhooks** and then **Add webhook**.
        
    * Enter the **Payload URL**:
        
    
    ```http
    http://<your-ec2-public-ip>:8080/github-webhook/
    ```
    
    * Set **Content type** to default one and enable **Just the push event**.
        
    * Click on **Add webhook** and wait for it to show a green tick, indicating successful setup.
        
    

---

### Step 14. Build the Project in Jenkins

1. **Trigger the First Build**:
    
    * Go back to the Jenkins dashboard and click on the **Build Now** button for your pipeline job.
        
    * This action will initiate the pipeline and deploy your Spring boot application.
        
2. **Access the Application**:
    
    * To allow incoming traffic to your application, go to your EC2 security group and add an inbound rule for **port 8000**.
        
    * After the build completes successfully, visit your deployed Spring boot application at:
        
    
    ```http
    http://<your-ec2-public-ip>:8000
    ```
    

---

### Step 15. Automatic Deployment

From this point on, any changes you make and push to the GitHub repository will automatically trigger Jenkins to run the pipeline, rebuild the Docker image, and redeploy the application. This completes the CI/CD setup for your Springboot bank Application.

---

## Conclusion

By following these steps, you have successfully set up a **CI/CD pipeline** to automate the deployment of your Springboot bank Application using **Jenkins**, **GitHub**, and **Docker, shared libraries, multinode agent etc**. This setup not only simplifies the deployment process but also enhances productivity by ensuring that every code change is automatically tested and deployed.
