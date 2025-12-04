pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/JuanCamiloGomezV/java-maven'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage("SonarQube Analysis") {
            environment {
                scannerHome = tool 'sonar-scanner'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=ci-demo \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target
                    """
                }
            }
        }

        stage("Tests") {
            steps {
                sh "mvn test"
            }
        }

        stage("Notifications") {
            steps {
                echo "Aquí luego integramos Slack si quieres 2"
            }
        }
    }
}
