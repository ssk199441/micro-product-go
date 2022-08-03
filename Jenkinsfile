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
                sh 'docker run gesellix/trufflehog --json https://github.com/cehkunal/webapp.git > trufflehog'
                sh 'cat trufflehog'
            }
        }
        stage("source-Composition-analysis") {
            steps {
                echo 'Source Composition Analysis Started'
                sh 'rm owasp* || true'               
                sh 'wget "https://raw.githubusercontent.com/ssk199441/micro-product-go/master/dependency-check.sh"'
                sh 'chmod +x owasp-dependency-check.sh'
                sh 'bash owasp-dependency-check.sh'
                sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/data/cache/dependency-check-report.xml'
            }
        }
        stage("unit-test") {
            steps {
                echo 'UNIT TEST EXECUTION STARTED'
                sh 'make unit-tests'
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
