# DevSecOps-SwiggyClone-AWS-ECS-BlueGreen-Pipeline
- DevSecOps CI/CD for Swiggy-Clone. Implements automated Blue-Green Deployment on AWS ECS using CodePipeline and CodeDeploy.
  <img width="802" height="348" alt="image" src="https://github.com/user-attachments/assets/33019516-ca31-4d1c-9875-d36fb77ef370" />

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

  