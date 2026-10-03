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

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'CI/CD pipeline failed.'
        }
    }
}
