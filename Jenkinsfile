pipeline {
    agent {
        label 'AGENT-1'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

    stages {
        stage('build') {
            steps {
                sh """
                 docker build -t frontend:v1.0.1 .
                 docker tag frontend:v1.0.1 202533543549.dkr.ecr.us-east-1.amazonaws.com/expense/dev/mysql:v1.0.1
                 aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 202533543549.dkr.ecr.us-east-1.amazonaws.com
                 docker push 202533543549.dkr.ecr.us-east-1.amazonaws.com/expense/dev/mysql:v1.0.1
                """
            }
        }

        stage('test') {
            steps {
                sh 'echo this is test'
                sh 'sleep 10'
            }
        }

        stage('deploy') {
            steps {
                sh """
                aws eks update-kubeconfig --region us-east-1 --name expense-dev
                kubectl apply -f manifest.yaml
                """
            }
        }
        
    }
}
