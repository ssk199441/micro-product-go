pipeline {

    agent any

    tools {
        go 'go1.17'
    }
    environment {
        GO117MODULE = 'on'
        CGO_ENABLED = 0 
        GOPATH = "${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${BUILD_ID}"
    }

    stages {
        stage("check-git-secrets") {
            steps {
                echo 'Checking Secrets'
                sh 'rm trufflehog || true'
                sh 'docker run gesellix/trufflehog --json https://github.com/ssk199441/micro-product-go.git > trufflehog'
                sh 'cat trufflehog'
            }
        }       
        stage("functional-test") {
            steps {
                echo 'FUNCTIONAL TEST EXECUTION STARTED'
                sh 'make functional-tests'
            }
        }
        stage("build") {
            steps {
                echo 'BUILD EXECUTION STARTED'
                sh 'go version'
                sh 'go get ./...'
                sh 'docker build . -t ssk1994/product-go-micro'
            }
        }
        stage("scan-docker") {
            steps {
                echo 'Docker Image Scanning Started'
                sh 'git clone https://github.com/docker/docker-bench-security.git'
                sh 'cd docker-bench-security`'
                sh 'docker-compose run --rm docker-bench-security'
            }
        }
        stage('Docker Push') {
            agent any
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerhubPassword', usernameVariable: 'dockerhubUser')]) {
                sh "docker login -u ${env.dockerhubUser} -p ${env.dockerhubPassword}"
                sh 'docker push ssk1994/product-go-micro'
                }
            }
        }
    }
}
