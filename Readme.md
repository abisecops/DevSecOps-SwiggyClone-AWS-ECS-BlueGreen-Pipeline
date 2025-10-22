# DevSecOps-SwiggyClone-AWS-ECS-BlueGreen-Pipeline
- DevSecOps CI/CD for Swiggy-Clone. Implements automated Blue-Green Deployment on AWS ECS using CodePipeline and CodeDeploy.
<img width="802" height="348" alt="image" src="https://github.com/user-attachments/assets/8530cb53-64c3-4472-b3c6-35706848cf35" />

## Introduction
- The foundation of modern development is speed and reliability. By adopting DevSecOps and the Blue-Green Deployment strategy on AWS ECS, we can achieve both, ensuring our Swiggy-Clone application updates are fast, secure, and downtime-free.

## Blue Green Deployment
- Blue-Green Deployment is a release technique where two identical production environments ('Blue' for the current version and 'Green' for the new version) are maintained. The new version is deployed to the idle 'Green' environment, tested, and then traffic is instantly or gradually switched from 'Blue' to 'Green', minimizing downtime and enabling fast rollback.


| Environment | Role in Deployment | Status (Typically) |
| :--- | :--- | :--- |
| **Blue** | Current Production: Serves all live customer traffic. | **Live/Active** |
| **Green** | New Release: Receives the new application version for testing. | **Staging/Idle** |

Here are the notes for setting up the **AWS ECS Blue-Green Deployment Pipeline**, focusing on the key services and stages:

## 🚀 AWS ECS Blue-Green Deployment Setup Notes

| AWS Service | Role | Core Action |
| :--- | :--- | :--- |
| **AWS ECS** | **Hosting Platform** | Highly scalable **container orchestration** service (where the Swiggy-Clone app lives). |
| **CodePipeline** | **CI/CD Orchestrator** | Automates the entire release process: **Source → Build → Deploy**. |
| **CodeBuild** | **Build & Test** | Takes source code, builds the **Docker image** (Swiggy-Clone container), and runs tests. |
| **CodeDeploy** | **Deployment Manager** | Manages the **Blue-Green strategy** and controls traffic flow via the ALB. |

***

## 1. ⚙️ CodePipeline Stages & Steps

| Stage | Action | Details |
| :--- | :--- | :--- |
| **1. Source Stage** | Connect Repository | Link CodePipeline to your code source (**GitHub**, CodeCommit, etc.). |
| | Trigger | Pipeline starts automatically when changes are pushed to the main branch. |
| **2. Build Stage** | Build Image | Use **CodeBuild** to compile code, run unit tests, and create the **Docker image**. |
| | Push Artifact | Push the new Docker image to **Amazon ECR** (Elastic Container Registry). |
| **3. Deploy Stage** | CodeDeploy Action | Trigger the Blue-Green deployment using **CodeDeploy for ECS**. |

***

## 2. 🚦 Blue-Green Mechanism (in Deploy Stage)

This is how **CodeDeploy** executes the zero-downtime switch:

-   **Environment Definition:** Two identical **ECS Services** are managed:
    -   **Blue:** The service running the **current live version**.
    -   **Green:** The service used for the **new deployment**.
-   **Deployment:** CodeDeploy deploys the new Swiggy-Clone version to the **Green service** first.
-   **Validation:** Automated tests run against the Green service (using a temporary listener on the **ALB**).
-   **Traffic Shift (ALB Routing):** If validation passes, CodeDeploy automates the **Application Load Balancer (ALB)** to shift customer traffic:
    -   Can be **instantaneous** (all at once) or **gradual** (linear/canary).
-   **Safety Net:** The Blue service is kept running temporarily, allowing for **instant rollback** if monitoring detects issues after the traffic switch.

  ## Steps 
  
### **Step 1: Create a Sonar Server**

### 1.1 Create AWS Key-Pair

  * **Navigate** to AWS Console \> **Key Pairs**.
  * Click **“Create key-pair”**.
  * **Provide a name** and select **.pem** file type.
  * Click **Create** (The `.pem` file will download).
    - sonar.pem
<img width="1916" height="746" alt="image" src="https://github.com/user-attachments/assets/5b8a51e9-f42b-48c6-98e6-8448644facf9" />


### 1.2 Launch EC2 Instance

  * **Navigate** to EC2 Console \> **Launch Instance**.
  * **Give a name** (e.g., `Sonar-Server`).
  * Select **AMI: Ubuntu** and **Instance Type: t2.medium**.
    <img width="1083" height="434" alt="image" src="https://github.com/user-attachments/assets/ad83e125-f054-454a-b7ec-010353deed5d" />

  * Under Key Pair, **select the key-pair created** in step 1.1.
    <img width="1852" height="862" alt="image" src="https://github.com/user-attachments/assets/65c312e8-d265-4eb1-aee2-fb1317f51319" />
  * Click **“Launch Instance”** (keep others default).
    <img width="1805" height="475" alt="image" src="https://github.com/user-attachments/assets/3e58dd93-9ce4-461c-8d6a-84fb0a95ee21" />


### 1.3 Connect and Install Docker

  * Once the instance is **Running**, select it and click **Connect**.
  * Connect using EC2 Instance Connect or SSH with the `.pem` file.
  * **Install Docker** (Execute the following commands):
    ```bash
    sudo apt update
    sudo apt install docker.io -y
    sudo usermod -aG docker ubuntu
    sudo systemctl restart docker
    sudo chmod 777 /var/run/docker.sock
    ```
<img width="1919" height="917" alt="image" src="https://github.com/user-attachments/assets/ab310ac6-ec4d-452d-9f2e-e1a2f3ba8fe9" />

### 1.4 Run SonarQube Container

  * **Run SonarQube** as a Docker container:
    ```bash
    docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
    ```
<img width="977" height="343" alt="image" src="https://github.com/user-attachments/assets/3949f921-356b-457f-8d7f-053ef891677e" />

### 1.5 Configure Security Group

  * **Ensure port 9000 is opened** in the Security Group associated with this EC2 instance (Inbound Rules: Custom TCP 9000, Source: My IP or 0.0.0.0/0).
<img width="1897" height="539" alt="image" src="https://github.com/user-attachments/assets/b02a5cd0-fca3-440e-adaa-89ce791f9f66" />

### 1.6 Access SonarQube

  * **Access SonarQube** via your browser: `http://<public_ip>:9000`
  * **Default Credentials:** `Username: admin`, `Password: admin`
    <img width="1256" height="476" alt="image" src="https://github.com/user-attachments/assets/2e98e656-acce-43dd-a73b-66df2d6b25d6" />
  * Change password old to new: 
  <img width="709" height="721" alt="image" src="https://github.com/user-attachments/assets/1e5b9225-fc7e-4f86-9def-dcc105294e78" />


  
