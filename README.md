# Java Web Application CI/CD Pipeline

## Project Overview

This project demonstrates an end-to-end Continuous Integration and Continuous Deployment (CI/CD) pipeline for a Java web application.

The application source code is stored in GitHub. A GitHub webhook automatically triggers Jenkins whenever new code is pushed. Jenkins retrieves the latest source code, builds and tests the application using Maven, generates a WAR file, and deploys the WAR file to Apache Tomcat.

## Architecture

```text
Developer
    |
    | Git Push
    v
GitHub Repository
    |
    | GitHub Webhook
    v
Jenkins Server
    |
    | Checkout
    | Maven Build
    | Automated Tests
    | WAR Generation
    v
WAR File
    |
    | Tomcat Manager API
    v
Apache Tomcat
    |
    v
Java Web Application
```

## Technologies Used

* Git
* GitHub
* Jenkins
* Java 21
* Maven
* Apache Tomcat 9
* AWS EC2
* GitHub Webhook
* WAR

## GitHub Repository

Repository:

```text
https://github.com/gazaladayma/MyApp.git
```

Branch:

```text
main
```

## Jenkins

Jenkins Pipeline job:

```text
MyApp-CI-CD
```

The Jenkins pipeline contains four stages:

1. Checkout
2. Build
3. Verify WAR
4. Deploy to Tomcat

## Maven Build

Maven is used to compile, test and package the Java application.

```bash
mvn clean package
```

The generated WAR file is:

```text
target/myweb-0.0.4.war
```

## Automated Tests

The Maven tests completed successfully:

```text
Tests run: 3
Failures: 0
Errors: 0
Skipped: 0
```

## Tomcat

Apache Tomcat 9 is used as the application server.

Application context:

```text
/myweb
```

Tomcat port:

```text
8080
```

Application URL:

```text
http://<Tomcat-IP>:8080/myweb/
```

## GitHub Webhook

A GitHub webhook is configured to automatically trigger Jenkins after a GitHub push.

The process is:

```text
Git Push
    ↓
GitHub
    ↓
GitHub Webhook
    ↓
Jenkins
    ↓
Maven Build
    ↓
Tests
    ↓
WAR Generation
    ↓
Tomcat Deployment
    ↓
Updated Application
```

## How to Test the CI/CD Pipeline

### 1. Modify the Application

Update the application webpage, for example:

```text
index.html
```

### 2. Commit the Changes

```bash
git add .
git commit -m "Update application webpage"
```

### 3. Push to GitHub

```bash
git push origin main
```

### 4. Jenkins Automatically Starts

The GitHub webhook sends a notification to Jenkins.

Jenkins automatically starts the pipeline.

### 5. Jenkins Builds the Application

Jenkins:

* Checks out the latest code.
* Runs Maven.
* Executes tests.
* Generates the WAR file.
* Verifies the WAR file.

### 6. Jenkins Deploys to Tomcat

The generated WAR file is automatically deployed to Tomcat using the Tomcat Manager API.

The application context is:

```text
/myweb
```

### 7. Verify the Application

Open:

```text
http://<Tomcat-IP>:8080/myweb/
```

Refresh the page and verify that the updated application content is displayed.

## Deployment Method

The WAR file is deployed using the Tomcat Manager API.

The deployment uses:

```text
path=/myweb
update=true
```

`update=true` allows the existing application to be updated with the newly generated WAR file.

## Jenkins Credentials

Tomcat deployment credentials are stored securely in Jenkins Credentials.

Credential ID:

```text
tomcat-jenkins-credentials
```

The password is not stored directly in the Jenkinsfile.

The pipeline uses Jenkins `withCredentials` to access the credentials securely.

## Jenkinsfile

The Jenkinsfile is stored in the root directory of the GitHub repository.

Pipeline flow:

```text
Checkout
    ↓
Build
    ↓
Verify WAR
    ↓
Deploy to Tomcat
```

## Expected Result

After a developer pushes a change to GitHub:

```text
GitHub
   ↓
Webhook
   ↓
Jenkins
   ↓
Maven
   ↓
WAR
   ↓
Tomcat
   ↓
Updated Web Application
```

The application is automatically rebuilt and redeployed.

## Project Result

The end-to-end CI/CD pipeline was successfully implemented and tested.

The pipeline successfully:

* Retrieved source code from GitHub.
* Built the Java application using Maven.
* Ran automated tests.
* Generated the WAR artifact.
* Deployed the WAR to Apache Tomcat.
* Automatically redeployed the application after a source-code update.
* Displayed the updated application on Tomcat.
