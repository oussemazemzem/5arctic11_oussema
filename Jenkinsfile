pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    environment {
        SONAR_PROJECT_KEY = '5arctic11_oussema-backend'
        DOCKERHUB_USER     = 'zamassuo'
        IMAGE_BACKEND      = "${DOCKERHUB_USER}/oussemazemzem-5arctic11-backend"
        IMAGE_FRONTEND     = "${DOCKERHUB_USER}/oussemazemzem-5arctic11-frontend"
    }

    stages {

        stage('Clone') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/oussemazemzem/5arctic11_oussema.git'
            }
        }

        stage('Build Backend (Maven)') {
            steps {
                dir('backend') {
                    sh 'mvn clean install package'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('backend') {
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.10.0.2594:sonar \
                              -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                              -Dsonar.token=\$SONAR_AUTH_TOKEN
                        """
                    }
                }
            }
        }

        stage('Build Frontend (Angular)') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }

        stage('Docker Build Images') {
            steps {
                sh "docker build -t ${IMAGE_BACKEND}:latest ./backend"
                sh "docker build -t ${IMAGE_FRONTEND}:latest ./frontend"
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${IMAGE_BACKEND}:latest"
                    sh "docker push ${IMAGE_FRONTEND}:latest"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès : build, analyse Sonar et images Docker publiées.'
        }
        failure {
            echo 'Le pipeline a échoué.'
        }
        always {
            sh 'docker logout || true'
        }
    }
}
