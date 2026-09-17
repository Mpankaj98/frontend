pipeline {
    agent any   // or docker { image '...' } / label '...'

    environment {
        DOCKER_IMAGE = "yourusername/myapp"
        // Add other env vars / credentials bindings here
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm   // preferred when job is configured with Git
                // or:
                // git branch: 'main',
                //     url: 'https://github.com/your-org/your-repo.git',
                //     credentialsId: 'github-creds'
            }
        }

        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {   // name of the SonarQube server config in Jenkins
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                    trivy fs \
                        --exit-code 1 \
                        --severity HIGH,CRITICAL \
                        .
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def image = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        image.push()
                        image.push('latest')   // optional
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {   // Jenkins SSH credentials ID
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@YOUR_EC2_IP '
                            docker pull ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            # docker stop/rm + run your container here
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        failure {
            // notify, etc.
        }
    }
}
