pipeline {
    agent { 
        label 'linux_ubuntu' 
    }

    environment {
        REGISTRY_HOST  = '172.18.50.164'
        REGISTRY_REPO  = '' 
        IMAGE_NAME     = 'spring-petclinic-image' 
        SHORT_COMMIT   = "${env.GIT_COMMIT ? env.GIT_COMMIT[0..6] : 'latest'}"
        REGISTRY_CREDS = 'nexus-repo-credentials-id' 
    }
   
    stages {
        stage('Checkstyle') {
	    when {
		not { branch 'main' }
	    }
            steps {
               echo 'Checkstyle started'
		withCredentials([usernamePassword(credentialsId: env.REGISTRY_CREDS, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                        sh "./gradlew checkstyleMain checkstyleTest --continue -PnexusUsername='${NEXUS_USER}' -PnexusPassword='${NEXUS_PASS}'"
                  }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'build/reports/checkstyle/**', allowEmptyArchive: true
                }
            }
        }
        
        stage('Test') {
	    when {
		not { branch 'main' }
	    }
            steps {
                echo 'Tests started'
		withCredentials([usernamePassword(credentialsId: env.REGISTRY_CREDS, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                        sh "./gradlew test -PnexusUsername='${NEXUS_USER}' -PnexusPassword='${NEXUS_PASS}'"
                  }
            }
        }
        
        stage('Build & Push Image') {
            steps {
                script {
		    def targetPort = ""
                    if (env.BRANCH_NAME == 'main') {
                        targetPort = "8086"
                        echo "MAIN BRANCH port: ${targetPort}"
                    } else {
                        targetPort = "8087"
                        echo "MERGE REQUEST port: ${targetPort}"
                    }
		    def currentRegistry = "${REGISTRY_HOST}:${targetPort}"
                    def commitHash = env.GIT_COMMIT ? env.GIT_COMMIT.take(7) : 'latest'
                    def fullImageTarget = "${currentRegistry}/${IMAGE_NAME}:${commitHash}"
                    
                    echo "Building Docker image: ${fullImageTarget}"
                    sh "docker build -t ${fullImageTarget} ."
                    
                    withCredentials([usernamePassword(credentialsId: env.REGISTRY_CREDS, usernameVariable: 'REGISTRY_USER', passwordVariable: 'REGISTRY_PASS')]) {
                        echo 'Log into Nexus: '
                        sh "docker login -u '${REGISTRY_USER}' -p '${REGISTRY_PASS}' ${currentRegistry}"    
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
