# DevOps CI/CD Pipeline Using Jenkins, GitHub, Maven & Tomcat

## Project Overview

This project demonstrates an end-to-end CI/CD pipeline for a Java web application.

The application source code is stored in GitHub. A GitHub webhook automatically triggers Jenkins whenever changes are pushed to the repository.

Jenkins checks out the latest source code, builds and tests the application using Maven, creates a WAR file, verifies the WAR artifact, and deploys the WAR file to an Apache Tomcat server using the Tomcat Manager API.

The complete process is automated from source-code change to application deployment.

## CI/CD Flow

```text
GitHub
   |
   | Push / Webhook
   v
Jenkins
   |
   v
Checkout
   |
   v
Maven Build & Test
   |
   v
WAR Artifact
   |
   v
Verify WAR
   |
   v
Tomcat Manager API
   |
   v
Apache Tomcat
   |
   v
Updated Web Application
```

## Technologies Used

GitHub – Source code repository
GitHub Webhook – Automatically triggers Jenkins
Jenkins – CI/CD automation
Maven – Java build, test and packaging tool
Java 21 – Java development environment
Apache Tomcat 9 – Application server
Amazon EC2 – Server infrastructure
Linux – Server operating system
WAR – Java web application package
Tomcat Manager API – Application deployment

## GitHub Repository

GitHub repository:

https://github.com/gazaladayma/MyApp

The repository contains the Java web application source code and Jenkinsfile.

## Application Details

The Maven project uses:

```text
Group ID:     in.javahome
Artifact ID:  myweb
Version:      0.0.4
Packaging:    WAR
```

The generated WAR file is:

```text
myweb-0.0.4.war
```

The application displays:

```text
Java Home Web Application!!!!
```

## Infrastructure

### Jenkins Server

A dedicated Amazon EC2 instance was used for Jenkins.

### Configuration

```text
Operating System: Amazon Linux 2023
Java:             21
Maven:            3.8.4
Git:              2.50.1
Jenkins:          2.580.1
Jenkins Port:     8080
```

Jenkins was accessed through the configured EC2 server address.

### Tomcat Server

A separate Amazon EC2 instance was used for Apache Tomcat.

### Configuration

```text
Operating System: Amazon Linux 2023
Java:             21
Tomcat:           9
Tomcat Port:      8080
```

The application was deployed to Tomcat using the Tomcat Manager API.

## Jenkins Pipeline

The Jenkins job is:

```text
MyApp-CI-CD
```

The pipeline contains four stages:

1. Checkout
2. Build
3. Verify WAR
4. Deploy to Tomcat

GitHub webhook integration automatically starts the pipeline after a push to the repository.

## Stage 1 — Checkout

Jenkins retrieves the latest source code from the GitHub repository.

```groovy
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

The Jenkins pipeline is configured to use the GitHub repository:

```text
https://github.com/gazaladayma/MyApp.git
```

## Stage 2 — Build

Jenkins uses Java 21 and Maven to build and test the application.

```groovy
stage('Build') {
    steps {
        sh '''
            export JAVA_HOME=/usr/lib/jvm/java-21-amazon-corretto.x86_64
            export PATH=$JAVA_HOME/bin:$PATH

            java -version
            mvn -version
            mvn clean package
        '''
    }
}
```

The Maven build performs the following:

* Compiles the Java application
* Runs the unit tests
* Packages the application as a WAR file

The successful build produced:

```text
Tests run: 3
Failures: 0
Errors: 0
Skipped: 0

BUILD SUCCESS
```

The generated WAR file is:

```text
target/myweb-0.0.4.war
```

## Stage 3 — Verify WAR

Jenkins verifies that the WAR file was successfully generated.

```groovy
stage('Verify WAR') {
    steps {
        sh 'ls -lh target/myweb-0.0.4.war'
    }
}
```

Example result:

```text
target/myweb-0.0.4.war
```

The WAR file was successfully created before deployment.

## Stage 4 — Deploy to Tomcat

Jenkins securely retrieves the Tomcat credentials from Jenkins Credentials and uses the Tomcat Manager API to deploy the WAR file.

```groovy
stage('Deploy to Tomcat') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'tomcat-jenkins-credentials',
            usernameVariable: 'TOMCAT_USER',
            passwordVariable: 'TOMCAT_PASSWORD'
        )]) {
            sh '''
                curl --fail --silent --show-error \
                -u "$TOMCAT_USER:$TOMCAT_PASSWORD" \
                --upload-file target/myweb-0.0.4.war \
                "http://172.31.47.69:8080/manager/text/deploy?path=/myweb&update=true"
            '''
        }
    }
}
```

The Tomcat deployment returned:

```text
OK - Deployed application at context path [/myweb]
```

The Tomcat application was then verified as running:

```text
/myweb:running:0:myweb
```

## Complete Jenkinsfile

The final Jenkins pipeline used for this project is:

```groovy
pipeline {

    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    export JAVA_HOME=/usr/lib/jvm/java-21-amazon-corretto.x86_64
                    export PATH=$JAVA_HOME/bin:$PATH

                    java -version
                    mvn -version
                    mvn clean package
                '''
            }
        }

        stage('Verify WAR') {
            steps {
                sh 'ls -lh target/myweb-0.0.4.war'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'tomcat-jenkins-credentials',
                    usernameVariable: 'TOMCAT_USER',
                    passwordVariable: 'TOMCAT_PASSWORD'
                )]) {
                    sh '''
                        curl --fail --silent --show-error \
                        -u "$TOMCAT_USER:$TOMCAT_PASSWORD" \
                        --upload-file target/myweb-0.0.4.war \
                        "http://172.31.47.69:8080/manager/text/deploy?path=/myweb&update=true"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'CI/CD pipeline failed.'
        }
    }
}
```

## Jenkins Credentials

The Tomcat username and password are stored securely in Jenkins Credentials.

Credential ID:

```text
tomcat-jenkins-credentials
```

The password is not stored directly in the Jenkinsfile.

Jenkins masks the password in the console output during deployment.

## GitHub Webhook

A GitHub webhook was configured to automatically notify Jenkins when changes are pushed to the repository.

The CI/CD process is therefore:

```text
Developer modifies code
        |
        v
Git commit
        |
        v
Git push
        |
        v
GitHub
        |
        | Webhook
        v
Jenkins
        |
        v
CI/CD Pipeline
```

This removes the need to manually click **Build Now** for every code change.

## Tomcat Deployment

The application is deployed to the Tomcat context:

```text
/myweb
```

The deployment was verified using the Tomcat Manager API.

The application was also tested directly from the server using:

```bash
curl -i http://172.31.47.69:8080/myweb/
```

The response returned the Java web application HTML successfully.

## Application Update Test

The CI/CD pipeline was tested using an actual webpage update.

### Step 1 — Original Application

The Java web application was running on Tomcat.

### Step 2 — Modify the Webpage

The `index.html` file was updated with new webpage content.

### Step 3 — Commit and Push

The updated code was committed and pushed to GitHub.

### Step 4 — GitHub Webhook

The GitHub push automatically triggered Jenkins.

### Step 5 — Jenkins Pipeline

Jenkins automatically:

* Checked out the updated code
* Built the application using Maven
* Ran the tests
* Created a new WAR file
* Verified the WAR file
* Deployed the updated WAR to Tomcat

### Step 6 — Verify Updated Application

The application was refreshed after deployment.

The updated webpage was displayed successfully.

This confirmed that a code change in GitHub automatically reached the running application through the CI/CD pipeline.

## Successful CI/CD Pipeline

The final pipeline flow is:

```text
GitHub
     |
     | Webhook
     v
Jenkins
     |
     v
Checkout
     |
     v
Build & Test with Maven
     |
     v
WAR File
     |
     v
Verify WAR
     |
     v
Tomcat Manager API
     |
     v
Apache Tomcat
     |
     v
Updated Web Application
```

Build result:

```text
Finished: SUCCESS
```

The WAR file was successfully built and deployed to Apache Tomcat.

## Project Evidence

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
32_Tomcat_Server_Java_21_Installed.png
34_Tomcat_Server_Started.png
35_Tomcat_Welcome_Page.png
36_Tomcat_Webapps_Directory.png
41_Tomcat_Webapps_Ready.png
43_Tomcat_WAR_Deployed.png
44_Tomcat_Application_Running.png
47_Final_CI_CD_Application.png
```

### Jenkins / Tomcat Deployment

```text
45_Jenkins_CI_CD_Deployment_Success.png
43_Tomcat_WAR_Deployed.png
44_Tomcat_Application_Running.png
47_Final_CI_CD_Application.png
```

### GitHub Webhook / Application Update

The final demonstration should show:

```text
GitHub index.html updated
        |
        v
Git push
        |
        v
GitHub Webhook
        |
        v
Jenkins automatically triggered
        |
        v
Build & Test
        |
        v
WAR created
        |
        v
Tomcat deployment
        |
        v
Updated webpage
```

## How to Reproduce the Project

### 1. Clone the Repository

```bash
git clone https://github.com/gazaladayma/MyApp.git
cd MyApp
```

### 2. Build the Application Locally

Make sure Java 21 and Maven are installed.

Run:

```bash
mvn clean package
```

The WAR file will be generated under:

```text
target/myweb-0.0.4.war
```

### 3. Configure Jenkins

Create a Jenkins Pipeline job named:

```text
MyApp-CI-CD
```

Configure the pipeline to use the Jenkinsfile in the repository.

Configure the GitHub webhook so that a push to the repository triggers Jenkins.

### 4. Configure Tomcat

Install Java 21 and Apache Tomcat 9 on the Tomcat server.

Start Tomcat:

```bash
cd ~/tomcat
./bin/startup.sh
```

Verify that Tomcat is listening on port 8080:

```bash
ss -lntp | grep 8080
```

### 5. Configure Tomcat Credentials

Create a Tomcat user with access to the Tomcat Manager application.

Store the username and password in Jenkins Credentials using:

```text
tomcat-jenkins-credentials
```

### 6. Run the CI/CD Pipeline

Make a change to the application and push it to GitHub:

```bash
git add .
git commit -m "Update web page"
git push origin main
```

The GitHub webhook automatically triggers Jenkins.

Jenkins will:

1. Checkout the latest code.
2. Build the application with Maven.
3. Run the tests.
4. Generate the WAR file.
5. Verify the WAR file.
6. Deploy the WAR to Tomcat.
7. Complete the pipeline.

### 7. Verify the Application

The application context is:

```text
/myweb
```

The application can be verified from the Tomcat server using:

```bash
curl http://172.31.47.69:8080/myweb/
```

The updated webpage should be displayed after the deployment completes.

## Result

The project successfully demonstrates an automated CI/CD workflow:

```text
Developer
    |
    v
GitHub Repository
    |
    | Webhook
    v
Jenkins
    |
    +----> Checkout
    |
    +----> Maven Build & Test
    |
    +----> WAR File
    |
    +----> Verify WAR
    |
    +----> Tomcat Deployment
              |
              v
        Apache Tomcat
              |
              v
      Updated Web Application
```

## Final Result

The Java web application was successfully:

* Stored in GitHub
* Automatically detected through a GitHub webhook
* Checked out by Jenkins
* Built and tested using Maven
* Packaged as a WAR file
* Verified by Jenkins
* Deployed to Apache Tomcat
* Updated automatically after a GitHub code change

**The complete CI/CD workflow successfully demonstrated automatic deployment from GitHub to a running Tomcat application.**
