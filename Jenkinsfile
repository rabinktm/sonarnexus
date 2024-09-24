pipeline {
    agent {
        label 'slave-node'
    }
    environment {
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

        stage('UNIT TEST') {
            steps {
                sh 'mvn -f pom.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -f pom.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar6.1') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=java-tomcat-sample \
                        -Dsonar.projectName=java-tomcat-sample \
                        -Dsonar.projectVersion=4.0 \
                        -Dsonar.sources=jenkins/java-tomcat-sample/src/ \
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
                         file: 'jenkins/java-tomcat-sample/target/java-tomcat-maven-example.war',
                         type: 'war']
                    ]
                )
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
            subject: 'BUILD SUCCESS NOTIFICATION',
            body: """Hello!,

Build #$BUILD_NUMBER was successful. Please review it at:
$BUILD_URL
Regards,
DevOps Team"""
        }
        failure {
            mail to: 'domain.nova@gmail.com',
            subject: 'BUILD FAILED NOTIFICATION',
            body: """Hello!,

Build #$BUILD_NUMBER failed. Please review it at:
$BUILD_URL
Regards,
DevOps Team"""
        }
    }
}
