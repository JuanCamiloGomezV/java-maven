pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {
        // Carpeta en tu Mac donde “desplegaremos producción”
        PROD_PATH = "/Users/$USER/production-ci-demo"
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

        stage("Tests") {
            steps {
                sh "mvn test"
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

        stage('Quality Gate') {
            steps {
                script {
                    // Espera hasta 1 minuto por la respuesta de SonarQube
                    def qg = waitForQualityGate(timeout: '1m')

                    if (qg.status != 'OK') {
                        error "❌ Quality Gate falló: ${qg.status}"
                    }
                }
            }
        }

        stage('Build Final Artifact') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Deploy to Production') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                script {
                    sh """
                        mkdir -p $PROD_PATH
                        cp target/*.jar $PROD_PATH/app.jar
                    """
                }
                echo "🚀 Despliegue exitoso en producción local: $PROD_PATH"
            }
        }

        stage("Notifications") {
            steps {
                echo "Aquí luego integramos Slack si quieres"
            }
        }
    }

    post {
        success {
            emailext (
                to: "tucorreo@correo.com",
                subject: "✔ Despliegue exitoso",
                body: "El código pasó SonarQube y se desplegó correctamente en producción."
            )
        }
        failure {
            emailext (
                to: "camilo12378@gmail.com",
                subject: "❌ Error en el pipeline",
                body: "El pipeline falló. Revisa la etapa correspondiente en Jenkins."
            )
        }
    }
}
