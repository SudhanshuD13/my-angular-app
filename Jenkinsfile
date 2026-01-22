pipeline {
    agent any 

    environment {
        DOCKER_USER = 'sudhanshud100'
        IMAGE_NAME  = 'my-angular-frontend'
        DOCKER_HUB_CREDS = 'docker-hub-creds'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Scanner tool wahi rahega jo Jenkins mein configure kiya tha
                    def scannerHome = tool 'sonar-scanner'
                    
                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=my-angular-app \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://sonarqube-server:9000 \
                        -Dsonar.login="sqa_4e04366daba8c1c9a0df6c5b99750c5944d2b859"\
                    }
                }
            }
        }

        stage('Docker Build & Tag') {
            steps {
                script {
                    echo "Building Angular Image..."
                    sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:v${env.BUILD_ID} ."
                    sh "docker tag ${DOCKER_USER}/${IMAGE_NAME}:v${env.BUILD_ID} ${DOCKER_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDS}", passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER_ENV')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER_ENV --password-stdin"
                        sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:v${env.BUILD_ID}"
                        sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }
        stage('Trigger Backend Deploy') {
            steps {
            // Ye Go wali pipeline ko trigger karega
            build job: 'my-go-pipeline', wait: false
            }
        }
    }
    

    // Post section hamesha chalta hai, chahe build success ho ya fail
    post {
        always {
            script {
                echo "Cleaning up local images..."
                // Purani images delete karna taaki space bache
                sh "docker rmi ${DOCKER_USER}/${IMAGE_NAME}:v${env.BUILD_ID}"
                sh "docker rmi ${DOCKER_USER}/${IMAGE_NAME}:latest"
                
                // Ye command un faltu layers ko saaf karegi jo build ke waqt banti hain (Dangling images)
                sh "docker image prune -f"
            }
        }
    }
}