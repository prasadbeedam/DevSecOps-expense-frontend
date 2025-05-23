pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    parameters{
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }
    environment{
        def appVersion = '' //variable declaration
        region = "us-east-1"
        account_id = "202533543549"
    }
    stages {
        stage('read the version'){
            steps{
                script{
                    ef packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version: $appVersion"
                }
            }
        }
        
        stage('Build'){
            steps{
                sh """
                zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                ls -ltr
                """
            }
        }
        stage('Docker build'){
            steps{
                sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

                    docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend:${appVersion} .

                    docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend:${appVersion}
                """
            }
        }

        stage('Deploy'){
            steps{
                sh """
                    aws eks update-kubeconfig --region us-east-1 --name expense
                    kubectl apply -f manifest.yaml
                """
            }
        }
   
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success { 
            echo 'I will run when pipeline is success'
        }
        failure { 
            echo 'I will run when pipeline is failure'
        }
    }
}