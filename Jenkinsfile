pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
        // Remove the sonarQube line if it still causes issues
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_USERNAME = 'gigaops'  // Set this directly to your Docker Hub username
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/spotify-app:latest"
        TARGET_VM = "root@192.168.82.111"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/GigaOpsSol/SonarQube-Project-Kastro.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Kastro -Dsonar.projectKey=KastroKey -Dsonar.java.binaries=target"
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn package"
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // Building Docker Image
                    sh "docker build -t $DOCKER_IMAGE ."
                }
            }
        }

        stage('Docker Push to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        echo "Docker Hub Username: ${DOCKERHUB_USERNAME}"  // Check the username
                        echo "Docker Image: ${DOCKER_IMAGE}"  // Check the image
                        sh "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"
                        sh "docker push $DOCKER_IMAGE"
                    }
                }
            }
        }

        stage('Deploy to On-Prem VM') {
            steps {
                sshagent(['deploy-vm-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no $TARGET_VM '
                            docker pull $DOCKER_IMAGE &&
                            docker stop spotify-app || true &&
                            docker rm spotify-app || true &&
                            docker run -d --name spotify-app -p 5555:5555 $DOCKER_IMAGE
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
