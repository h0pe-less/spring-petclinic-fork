pipeline {
    agent { 
        label 'linux_ubuntu' 
    }

    environment {
        REGISTRY_URL   = '172.18.50.164:8087'
        REGISTRY_REPO  = '' 
        IMAGE_NAME     = 'spring-petclinic-image' 
        SHORT_COMMIT   = "${env.GIT_COMMIT ? env.GIT_COMMIT[0..6] : 'latest'}"
        REGISTRY_CREDS = 'nexus-repo-credentials-id' 
    }
   
    stages {
        stage('Checkstyle') {
            steps {
               echo 'Checkstyle started'
                sh './gradlew checkstyleMain checkstyleTest --continue'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'build/reports/checkstyle/**', allowEmptyArchive: true
                }
            }
        }
        
        stage('Test') {
            steps {
                echo 'Tests started'
                sh './gradlew test'
            }
        }
        
        stage('Build & Push Image') {
            steps {
                script {
                    def fullImageTarget = "${REGISTRY_URL}/${IMAGE_NAME}:${SHORT_COMMIT}"
                    
                    echo "Building Docker image: ${fullImageTarget}"
                    sh "docker build -t ${fullImageTarget} ."
                    
                    withCredentials([usernamePassword(credentialsId: env.REGISTRY_CREDS, usernameVariable: 'REGISTRY_USER', passwordVariable: 'REGISTRY_PASS')]) {
                        echo 'Log into Nexus: '
                        sh "docker login -u '${REGISTRY_USER}' -p '${REGISTRY_PASS}' ${REGISTRY_URL}"
                        
                        echo 'Push into Nexus: '
                        sh "docker push ${fullImageTarget}"
                    }
                    
                    echo 'Clean up'
                    sh "docker rmi ${fullImageTarget}"
                }
            }
        }
    }
}
