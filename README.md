# DevOps CI/CD Pipeline Using Jenkins, GitHub, Maven & Tomcat

## Project Overview

This project demonstrates an end-to-end **CI/CD pipeline** for a Java web application.

The application source code is stored in GitHub. Jenkins automatically checks out the source code, builds the application using Maven, creates a WAR file, archives the WAR artifact, and deploys the WAR file to an Apache Tomcat server using SSH.

### CI/CD Flow

```text
GitHub
   |
   v
Jenkins
   |
   v
Maven Build
   |
   v
WAR Artifact
   |
   v
Jenkins Archive
   |
   v
SSH / SCP
   |
   v
Apache Tomcat
   |
   v
Running Web Application
```

---

## Technologies Used

* **GitHub** – Source code repository
* **Jenkins** – CI/CD automation
* **Maven** – Java build and packaging tool
* **Java 21** – Java development environment
* **Apache Tomcat 9** – Application server
* **Amazon EC2** – Jenkins and Tomcat servers
* **Amazon Elastic IP** – Stable public IP addresses
* **SSH / SCP** – Secure communication and WAR deployment
* **Linux** – Server operating system

---

## GitHub Repository

GitHub repository:

**https://github.com/gazaladayma/MyApp**

The repository is forked from:

**ARAVINDTrainings/MyApp**

---

## Application Details

The Maven project uses:

```text
Group ID:    in.javahome
Artifact ID: myweb
Version:     0.0.4
Packaging:   WAR
```

The generated WAR file is:

```text
myweb-0.0.4.war
```

The application displays:

```text
Java Home Web Application!!!!
```

---

# Infrastructure

## Jenkins Server

A dedicated Amazon EC2 instance was created for Jenkins.

### Configuration

```text
Server Name: Devops-Jenkins-Server
Operating System: Amazon Linux 2023
Instance Type: t2.micro
Java: 21
Maven: 3.8.4
Git: 2.50.1
Jenkins: 2.580.1
Jenkins Port: 8080
```

Jenkins was configured and accessed through:

```text
http://3.222.123.234:8080
```

---

## Tomcat Server

A separate Amazon EC2 instance was created for Apache Tomcat.

### Configuration

```text
Server Name: Devops-Tomcat-Server
Operating System: Amazon Linux 2023
Instance Type: t2.micro
Java: 21
Tomcat: 9.0.112
Tomcat Port: 8080
```

Tomcat was installed under:

```text
/home/ec2-user/tomcat
```

The Tomcat application deployment directory is:

```text
/home/ec2-user/tomcat/webapps/
```

---

# Jenkins Pipeline

The Jenkins job is:

```text
MyApp-CI-CD
```

The pipeline contains four stages:

```text
1. Checkout
2. Build with Maven
3. Archive WAR
4. Deploy to Tomcat
```

---

## Stage 1 — Checkout

Jenkins checks out the `master` branch from the user's GitHub repository.

```groovy
stage('Checkout') {
    steps {
        git branch: 'master',
            url: 'https://github.com/gazaladayma/MyApp.git'
    }
}
```

---

## Stage 2 — Build with Maven

Jenkins uses Java 21 and Maven to build the application.

```groovy
stage('Build with Maven') {
    steps {
        sh '''
            export JAVA_HOME=/usr/lib/jvm/java-21-amazon-corretto.x86_64
            export PATH=$JAVA_HOME/bin:$PATH

            echo "JAVA_HOME=$JAVA_HOME"
            java -version
            javac -version
            mvn -version

            mvn clean package
        '''
    }
}
```

The Maven build generates:

```text
target/myweb-0.0.4.war
```

---

## Stage 3 — Archive WAR

Jenkins archives the generated WAR file as a build artifact.

```groovy
stage('Archive WAR') {
    steps {
        archiveArtifacts artifacts: 'target/*.war',
            fingerprint: true
    }
}
```

The archived artifact is:

```text
myweb-0.0.4.war
```

---

## Stage 4 — Deploy to Tomcat

Jenkins uses the dedicated SSH key to securely copy the WAR file to the Tomcat server.

```groovy
stage('Deploy to Tomcat') {
    steps {
        sh '''
            scp -i /var/lib/jenkins/.ssh/id_ed25519 \
                -o StrictHostKeyChecking=no \
                target/*.war \
                ec2-user@100.57.250.146:/home/ec2-user/tomcat/webapps/
        '''
    }
}
```

Tomcat automatically detects the WAR file in the `webapps` directory and extracts it.

---

# Complete Jenkinsfile

The final Jenkins pipeline used for this project is:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/gazaladayma/MyApp.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh '''
                    export JAVA_HOME=/usr/lib/jvm/java-21-amazon-corretto.x86_64
                    export PATH=$JAVA_HOME/bin:$PATH

                    echo "JAVA_HOME=$JAVA_HOME"
                    java -version
                    javac -version
                    mvn -version

                    mvn clean package
                '''
            }
        }

        stage('Archive WAR') {
            steps {
                archiveArtifacts artifacts: 'target/*.war',
                    fingerprint: true
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                    scp -i /var/lib/jenkins/.ssh/id_ed25519 \
                        -o StrictHostKeyChecking=no \
                        target/*.war \
                        ec2-user@100.57.250.146:/home/ec2-user/tomcat/webapps/
                '''
            }
        }
    }
}
```

---

# SSH Configuration

A dedicated ED25519 SSH key was created on the Jenkins server for Jenkins-to-Tomcat communication.

The private key is stored on the Jenkins server at:

```text
/var/lib/jenkins/.ssh/id_ed25519
```

The corresponding public key was added to the Tomcat server:

```text
/home/ec2-user/.ssh/authorized_keys
```

The Jenkins service account was tested successfully using:

```bash
sudo -u jenkins ssh \
-i /var/lib/jenkins/.ssh/id_ed25519 \
ec2-user@100.57.250.146
```

The connection successfully reached the Tomcat server.

---

# Tomcat Deployment

After Jenkins copies the WAR file, Tomcat automatically extracts it.

The deployed files are:

```text
/home/ec2-user/tomcat/webapps/myweb-0.0.4.war
/home/ec2-user/tomcat/webapps/myweb-0.0.4/
```

The application context path is:

```text
/myweb-0.0.4
```

---

# Application URL

The final application is available at:

```text
http://100.57.250.146:8080/myweb-0.0.4/
```

The application displays:

```text
Java Home Web Application!!!!
```

---

# Successful CI/CD Pipeline

Jenkins Build #4 successfully completed all stages:

```text
Checkout
     |
     v
Build with Maven
     |
     v
Archive WAR
     |
     v
Deploy to Tomcat
     |
     v
Application Running
```

Build result:

```text
Finished: SUCCESS
```

The WAR file was successfully copied to the Tomcat server and deployed.

---

# Project Evidence

The following screenshots document the project setup, build, deployment, and final result.

### GitHub

```text
01_GitHub_Forked_Repository.png
03_Git_Clone_Success.png
04_Git_Repository_Verification.png
49_GitHub_Final_Repository.png
```

### Maven / Application Build

```text
06_Maven_POM_File.png
12_Maven_Build_Success.png
13_WAR_File_Generated.png
```

### Jenkins Server

```text
14_EC2_Jenkins_Server_Configuration.png
15_EC2_Jenkins_Server_Running.png
16_Elastic_IP_Associated.png
18_Jenkins_Server_Java_Installed.png
19_Jenkins_Server_Git_Installed.png
20_Jenkins_Server_Maven_Installed.png
21_Jenkins_Service_Running.png
22_Jenkins_Unlock_Page.png
23_Jenkins_Create_Admin_User.png
24_Jenkins_Dashboard.png
25_Jenkins_Tools_Page.png
26_Jenkins_Tools_Configured.png
```

### Jenkins Build

```text
27_Jenkins_Build_Success.png
28_Jenkins_WAR_Artifact.png
45_Jenkins_CI_CD_Deployment_Success.png
48_Jenkins_Final_Build_Summary.png
50_Jenkins_Pipeline_Stages_Success.png
51_Jenkins_WAR_Artifact_Final.png
```

### Tomcat Server

```text
29_EC2_Tomcat_Server_Running.png
30_Elastic_IP_Associated_Tomcat.png
31_Tomcat_Server_Updated.png
32_Tomcat_Server_Java_21_Installed.png
33_Tomcat_Extracted.png
34_Tomcat_Server_Started.png
35_Tomcat_Welcome_Page.png
36_Tomcat_Webapps_Directory.png
41_Tomcat_Webapps_Ready.png
43_Tomcat_WAR_Deployed.png
44_Tomcat_Application_Running.png
47_Final_CI_CD_Application.png
```

### Jenkins-to-Tomcat Deployment

```text
37_Tomcat_Jenkins_SSH_Key.png
38_Jenkins_To_Tomcat_SSH_Test.png
40_Jenkins_User_To_Tomcat_SSH.png
42_WAR_Copied_To_Tomcat.png
46_Tomcat_WAR_Automatic_Deployment.png
```

---

# How to Reproduce the Project

## 1. Clone the repository

```bash
git clone https://github.com/gazaladayma/MyApp.git
cd MyApp
```

## 2. Build the application locally

Make sure Java 21 and Maven are installed.

Run:

```bash
mvn clean package
```

The WAR file will be generated under:

```text
target/myweb-0.0.4.war
```

## 3. Configure Jenkins

Create a Jenkins Pipeline job named:

```text
MyApp-CI-CD
```

Configure the pipeline using the Jenkinsfile in this project.

## 4. Configure Tomcat

Install Java 21 and Apache Tomcat 9 on the Tomcat server.

Start Tomcat:

```bash
cd ~/tomcat
./bin/startup.sh
```

Verify port 8080:

```bash
ss -lntp | grep 8080
```

## 5. Configure SSH

Create an SSH key on the Jenkins server and add the public key to the Tomcat server's:

```text
~/.ssh/authorized_keys
```

Test the connection:

```bash
sudo -u jenkins ssh \
-i /var/lib/jenkins/.ssh/id_ed25519 \
ec2-user@<TOMCAT_IP>
```

## 6. Run Jenkins

Click:

```text
MyApp-CI-CD → Build Now
```

Jenkins will:

1. Checkout the GitHub repository.
2. Build the application with Maven.
3. Generate the WAR file.
4. Archive the WAR artifact.
5. Copy the WAR to Tomcat.

## 7. Verify the application

Open:

```text
http://<TOMCAT_IP>:8080/myweb-0.0.4/
```

---

# Result

The project successfully demonstrates an automated CI/CD workflow:

```text
Developer
    |
    v
GitHub Repository
    |
    v
Jenkins
    |
    +----> Checkout
    |
    +----> Maven Build
    |
    +----> WAR Artifact
    |
    +----> Archive Artifact
    |
    +----> SSH/SCP Deployment
              |
              v
        Apache Tomcat
              |
              v
       Java Web Application
```

**Final Result: Jenkins Build #4 — SUCCESS**

The Java web application was successfully built, packaged as a WAR, automatically deployed to Apache Tomcat, and accessed through the web browser.
