pipeline {

    triggers {
        pollSCM('H/5 * * * *')
    }

    agent any

    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }

        stage('compile and test') {
            agent {
               docker {
                   image 'maven:3.5.0-jdk-8-slim'
                   args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package'
                stash includes: 'target/*jar', name: 'binaries'
            }
        }

        stage('image') {
            steps {
                unstash 'binaries'
                sh "docker build -t jazapp/password-api:taller\${BUILD_ID} . --build-arg JAR_FILE=./target/passwordapi-1.5.2.jar"
            }
        }
        stage('publish') {
            steps {
                sh "docker push jazminsofiaf/password-api:taller\${BUILD_ID}"
            }
        }
        stage('deploy') {
            steps {
                sh "kubectl set image deployment/passnico passnico=jazapp/password-api:taller\${BUILD_ID}"
                sh "sleep 60"
            }
        }
        stage('acceptance_test') {
            steps {
                sh 'curl http://10.245.194.137/actuator/health'
            }
        }        
    }
}

