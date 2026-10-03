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
                step([
                    $class: 'hudson.plugins.deploy.DeployPublisher',
                    adapters: [[
                        $class: 'hudson.plugins.deploy.tomcat.Tomcat9xAdapter',
                        url: 'http://172.31.47.69:8080',
                        credentialsId: 'tomcat-jenkins-credentials',
                        alternativeDeploymentContext: '',
                        path: '/manager/text'
                    ]],
                    war: 'target/myweb-0.0.4.war',
                    contextPath: 'myweb'
                ])
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
