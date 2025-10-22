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
  


### 2.1 SonarQube Initial Configuration

1.  **Log in** to SonarQube (`<public_ip>:9000`) with default credentials (`admin`/`admin`).
    <img width="1256" height="476" alt="image" src="https://github.com/user-attachments/assets/2e98e656-acce-43dd-a73b-66df2d6b25d6" />
3.  **Create a custom password** when prompted.
    <img width="709" height="721" alt="image" src="https://github.com/user-attachments/assets/1e5b9225-fc7e-4f86-9def-dcc105294e78" />
5.  Click **“manually”** (`<>`) to create a new project.
   <img width="1478" height="744" alt="image" src="https://github.com/user-attachments/assets/b0d5018e-7af3-47d9-9b3f-c4a569fec93a" />

7.  Click **“locally”**.
    <img width="1460" height="769" alt="image" src="https://github.com/user-attachments/assets/31fb73b8-6a58-4a74-a001-143267efa2bf" />

9.  **Provide a name** for your project.
    
11.  Click **“Set Up”**.
   <img width="1386" height="628" alt="image" src="https://github.com/user-attachments/assets/18cacb89-ea2e-495a-8c68-3f186a37576c" />

### 2.2 Generate Sonar Token

1.  Click **“Generate”** to create a new token and **copy** it.
<img width="1401" height="494" alt="image" src="https://github.com/user-attachments/assets/c1071bd1-56a9-4df5-b508-895565e5cec1" />

3.  Under code, select **Other** and OS as **Linux**.
<img width="1422" height="917" alt="image" src="https://github.com/user-attachments/assets/4e1a5fba-de79-477e-9f55-1f57667ca001" />

### 2.3 Store Secrets in AWS Parameter Store

1.  In the AWS Console, navigate to **Systems Manager** $\rightarrow$ **Parameter Store**.
   <img width="1920" height="799" alt="image" src="https://github.com/user-attachments/assets/34850703-f314-4c15-a9f6-b37145da7493" />

3.  Click **“Create parameter”**.
   <img width="1918" height="868" alt="image" src="https://github.com/user-attachments/assets/c233c74b-8d5a-420d-bc1d-a76d1ed069cc" />

4.  **Create the Sonar Token parameter:**
      * **Name:** `/cicd/sonar/sonar-token`
      * **Value:** `<The copied Sonar token>`
        <img width="1896" height="923" alt="image" src="https://github.com/user-attachments/assets/89b93408-fa43-424e-bf65-4433cba46cf4" />

5.  **Create parameters for Docker Credentials:**
      * **Name:** `/cicd/docker-credentials/username`
      * **Value:** `<Your Docker Hub Username>`
      * **Name:** `/cicd/docker-credentials/password`
      * **Value:** `<Your Docker Hub Password>`
      * **Name:** `/cicd/docker-registry/url`
      * **Value:** `docker.io` (or your registry URL, could be ECR, Harbor)
        <img width="1920" height="586" alt="image" src="https://github.com/user-attachments/assets/cc7ebee3-5089-41a0-8686-6cdec6a35e9b" />

## **Step 3: Create AWS CodeBuild Project**

### 3.1 CodeBuild Project Creation

1.  **Navigate** to AWS **CodeBuild** $\rightarrow$ **Create project**.
   <img width="1920" height="783" alt="image" src="https://github.com/user-attachments/assets/f1a0549b-6a3d-4272-b266-a33d2ed5fe9b" />

2.  **Provide a name** for the project.
3.  Under **Source**, select **GitHub**.
4.  Select **Connect using OAuth** and follow the GitHub login/permission prompts.
<img width="1229" height="916" alt="image" src="https://github.com/user-attachments/assets/8c37a562-6d38-4d50-bc3b-a3e6b455ad8a" />

5.  Under **GitHub repo**, select your **application repository**.
<img width="644" height="406" alt="image" src="https://github.com/user-attachments/assets/9f8b989b-1dfe-45ea-99e0-274c556dff3f" />

6.  Under **Environment**, leave defaults (e.g., Managed Image, Ubuntu).
<img width="672" height="744" alt="image" src="https://github.com/user-attachments/assets/4c829fab-89d6-452f-9c1f-d8d6ec15b745" />

7.  Under **Buildspec**, select **“Use a buildspec file”**.
8.  **File Name:** Enter `buildspec.yaml`.
<img width="753" height="385" alt="image" src="https://github.com/user-attachments/assets/797b5e42-9ed7-4ab7-a3ef-390083e4e7ec" />
- project/swiggy-clone/buildspec.yaml

9.  Under **Artifacts**, select **Amazon S3** and use an **already created S3 bucket**.
<img width="798" height="782" alt="image" src="https://github.com/user-attachments/assets/b7a00eff-a121-403f-8142-2babf3c057be" />

10. Click **“Update project”** (or **Create project** if starting fresh).
<img width="755" height="468" alt="image" src="https://github.com/user-attachments/assets/8345df54-d709-4e91-b546-35cfcd8ff5a2" />


### 3.2 Update IAM Role Permissions

1.  **Navigate** to **IAM** $\rightarrow$ **Roles**.
2.  Find and click on the **Service Role** created by CodeBuild (e.g., `codebuild-<project-name>-service-role`).
<img width="1906" height="902" alt="image" src="https://github.com/user-attachments/assets/0cd0e4db-1930-4497-808b-5ff4a41206fa" />

3.  **Attach Policies** to grant necessary permissions:
      * **`AmazonSSMFullAccess`** (To access the Sonar/Docker parameters in Systems Manager).
        <img width="1808" height="559" alt="image" src="https://github.com/user-attachments/assets/06b8f0cb-a5ec-45db-a849-cffae0e36fe8" />

      * **`AWSS3FullAccess`** (To upload build artifacts to S3).
<img width="1617" height="574" alt="image" src="https://github.com/user-attachments/assets/a58ed775-f238-43b0-a168-e1159e2d9b08" />

### 3.3 Start Build

1.  In the CodeBuild project, click **“Start build”**.
<img width="1918" height="758" alt="image" src="https://github.com/user-attachments/assets/59eba47c-2f1e-4a38-8026-d50bbe644984" />

> **Note:** The `buildspec.yaml` file must be updated with your **Sonar URL** and **Project Key** before starting the build.

## Error Handling
- Sorry guys, I got an error:
<img width="1916" height="790" alt="image" src="https://github.com/user-attachments/assets/00aceb97-b2bb-4c3f-b4e8-3f59dd2ed7ea" />

Buildspec filename is wrong here from our repo, did you notice?
Let's change now and build again.

```project/swiggy-clone/buildspec.yaml``` to ```projects/Swiggy_clone/buildspec.yaml```
