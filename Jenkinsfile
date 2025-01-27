pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "novrian6/php_docker_app_image"
        DOCKER_TAG = "v1.0"
        APP_PORT = "8093"
        SONAR_SERVER = "https://sonarqube.invinsense.io/"
        TRIVY_SCAN_THRESHOLD = 10 // Threshold for vulnerabilities in Trivy
        DEPLOY_SERVER = "ec2-user@your-ec2-server-ip"
        DEPLOY_PATH = "/var/www/html"
        PEM_FILE = "/path/to/your/key.pem" 
        SONAR_PROJECT_KEY = 'Testing-project'       
        SONAR_PROJECT_NAME = 'Testing-project'      
        SONAR_HOST_URL = 'https://sonarqube.invinsense.io/' 
        SONAR_LOGIN = 'sqp_3893f7603eed42f3557afd52035b33319e343686' 
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    echo "Cloning repository from GitHub..."
                    git url: 'https://github.com/Sandiptank70/php_docker_app.git', branch: 'main'

                    echo "Listing workspace contents for verification..."
                    sh 'ls -la'
                }
            }
        }

        stage('SonarQube Scan') {
            stage('SonarQube Scan') 
        {
            steps 
            {
                withSonarQubeEnv('sonar-server') 
                { // Name of the SonarQube server in Jenkins config
                    script {
                        echo "Running SonarQube analysis..."
                        sh '''
                            sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_LOGIN}
                        '''
                        }
                }
            }        
        }}

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}"
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    echo "Scanning Docker image with Trivy..."
                    def trivyResult = sh(script: "trivy image --exit-code 1 --severity HIGH ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}", returnStatus: true)

                    if (trivyResult != 0) {
                        error "Trivy scan found vulnerabilities exceeding the threshold. Stopping pipeline."
                    } else {
                        echo "Trivy scan passed. No high-severity vulnerabilities found."
                    }
                }
            }
        }

        // stage('Push Docker Image to Docker Hub') {
        //     steps {
        //         script {
        //             echo "Pushing Docker image to Docker Hub: ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}"
        //             sh "docker push ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}"
        //         }
        //     }
        // }

        // stage('Deploy to EC2') {
        //     steps {
        //         script {
        //             echo "Deploying application to EC2 server..."
        //             sh "scp -i ${PEM_FILE} -o StrictHostKeyChecking=no docker-compose.yml ${DEPLOY_SERVER}:${DEPLOY_PATH}"
        //             sh "ssh -i ${PEM_FILE} -o StrictHostKeyChecking=no ${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && docker-compose up -d'"
        //         }
        //     }
        // }

        // stage('Approval for Tester') {
        //     steps {
        //         input message: 'Approve deployment?', ok: 'Approve'
        //     }
        // }

        // stage('Push Image with New Tag') {
        //     steps {
        //         script {
        //             def newTag = "approved-${DOCKER_TAG}"
        //             echo "Tagging Docker image with new tag: ${newTag}"
        //             sh "docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_TAG} ${DOCKER_IMAGE_NAME}:${newTag}"
                    
        //             echo "Pushing new tag to Docker Hub..."
        //             sh "docker push ${DOCKER_IMAGE_NAME}:${newTag}"
        //         }
        //     }
        // }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Please check the logs."
        }
    }
}


