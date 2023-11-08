pipeline {
    agent any
    tools {
        jdk "jdk17"
        nodejs "node16"
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Jaro233/Tetris-V1.git'
            }
        }
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=tetris-v1 \
                    -Dsonar.projectKey=tetris-v1 '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('NPM install') {
            steps {
                sh "npm install"
            }
        }
        stage('Trivy FS') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('OWASP FS') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker build and push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-login', toolName: 'docker') {
                        sh """
                        docker build -t j4ro123/tetrisv1 .
                        docker push j4ro123/tetrisv1
                        """
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image j4ro123/tetrisv1 > trivyImage.txt"
            }
        }
    }
}
