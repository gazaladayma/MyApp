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
                deploy(
                    adapters: [
                        tomcat9(
                            url: 'http://18.212.109.178:8080',
                            credentialsId: 'tomcat-jenkins-credentials'
                        )
                    ],
                    contextPath: 'myweb',
                    war: 'target/myweb-0.0.4.war'
                )
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
