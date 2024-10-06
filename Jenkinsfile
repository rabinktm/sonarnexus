pipeline {
    agent {
        label 'slave-node'
    }
    environment {
        DOCKER_HUB_REPO = "rabinktm/newrepo"
        DOCKER_CREDENTIALS_ID = "docker_credentials"
        scannerHome = tool 'sonar6.1'
    }
    stages {
        stage('Build Application') {
            steps {
                sh 'mvn -f pom.xml install -DskipTests'
            }
            post {
                success { 
                    echo "Now Archiving the Artifacts ..."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_HUB_REPO}:${BUILD_NUMBER} .'
            }
        }
        stage('Sonar Analysis') {
            
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=java-tomcat-sample \
                        -Dsonar.projectName=java-tomcat-sample \
                        -Dsonar.projectVersion=4.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage("UploadArtifact") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '192.168.44.11:8081',
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: 'trial',
                    credentialsId: 'nexusloginid',
                    artifacts: [
                        [artifactId: 'java-tomcat-sample',
                         classifier: '',
                         file: 'target/java-tomcat-maven-example.war',
                         type: 'war']
                    ]
                )
            }
        }
        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh 'echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin'
                }
            }
        } 
        stage('Push Docker Image') {
            steps {
                
                copyArtifacts filter: '**/*.war', fingerprintArtifacts: true, projectName: env.JOB_NAME, selector: specific(env.BUILD_NUMBER)
                echo "Building docker image"
                    sh '''
                    original_pwd=$(pwd -P)
                    docker push "${DOCKER_HUB_REPO}:${BUILD_NUMBER}"
                    docker rmi "${DOCKER_HUB_REPO}:${BUILD_NUMBER}"
                    '''
                }
            }
        stage('Deploy to Staging') {
            agent {
                label 'ubuntu-slave'
                  }
            steps{
                echo "Running app on staging env"
                 sh '''
                docker stop tomcatInstanceStaging || true
                docker rm tomcatInstanceStaging || true
                docker run -itd --name tomcatInstanceStaging -p 8082:8080 "${DOCKER_HUB_REPO}":$BUILD_NUMBER
                '''
            }
        }
    }
    
    post { 
        always { 
            mail to: 'domain.nova@gmail.com',
            subject: "Job '${JOB_NAME}' (${BUILD_NUMBER}) is waiting for input",
            body: "Please go to ${BUILD_URL} and verify the build"
        }
        success {
            mail to: 'domain.nova@gmail.com',
            subject: 'Build success notification',
            body: """Hello!,

Build #$BUILD_NUMBER was successful. Please review it at:
$BUILD_URL
Regards,
DevOps Team"""
        }
        failure {
            mail to: 'domain.nova@gmail.com',
            subject: 'Build failed notification',
            body: """Hello!,

Build #$BUILD_NUMBER failed. Please review it at:
$BUILD_URL
Regards,
DevOps Team"""
        }
    }
}
