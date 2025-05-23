pipeline { 
    agent {
        label 'AGENT-1'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

    parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }

    environment {
        def appVersion = ''
        region = "us-east-1"
        account_id = "202533543549"
    }

    stages {
        stage('Secret Scan') {
            steps {
                sh 'gitleaks detect --source . --verbose || true'
            }
        }

        stage('Read Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Application version: ${appVersion}"
                }
            }
        }

        stage('Install Dependencies & Audit') {
            steps {
                sh """
                    npm install
                    npm audit --audit-level=high || true
                    echo "Installed and scanned dependencies"
                """
            }
        }

        stage('Build') {
            steps {
                sh """
                    zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
                """
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

                    docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-backend:${appVersion} .

                    docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-backend:${appVersion}
                """
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh """
                    trivy image --exit-code 0 --severity HIGH,CRITICAL ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-backend:${appVersion} || true
                """
            }
        }

        stage('K8s Manifest Lint') {
            steps {
                sh 'kube-linter lint manifest.yaml || true'
            }
        }

        stage('Deploy') {
            when {
                expression { params.deploy }
            }
            steps {
                sh """
                    aws eks update-kubeconfig --region ${region} --name expense
                    kubectl apply -f manifest.yaml
                """
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            deleteDir()
        }
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}








// pipeline {
//     agent {
//         label 'AGENT-1'
//     }
//     options {
//         timeout(time: 30, unit: 'MINUTES')
//         disableConcurrentBuilds()
//         ansiColor('xterm')
//     }
//     parameters{
//         booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
//     }
//     environment{
//         def appVersion = '' //variable declaration
//         region = "us-east-1"
//         account_id = "202533543549"
//     }
//     stages {
//         stage('read the version'){
//             steps{
//                 script{
//                     def packageJson = readJSON file: 'package.json'
//                     appVersion = packageJson.version
//                     echo "application version: $appVersion"
//                 }
//             }
//         }
//         stage('Install Dependencies') {
//             steps {
//                sh """
//                 npm install
//                 ls -ltr
//                 echo "application version: $appVersion"
//                """
//             }
//         }
//         stage('Build'){
//             steps{
//                 sh """
//                 zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
//                 ls -ltr
//                 """
//             }
//         }
//         stage('Docker build'){
//             steps{
//                 sh """
//                     aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

//                     docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-backend:${appVersion} .

//                     docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-backend:${appVersion}
//                 """
//             }
//         }

//         stage('Deploy'){
//             steps{
//                 sh """
//                     aws eks update-kubeconfig --region us-east-1 --name expense
//                     kubectl apply -f manifest.yaml
//                 """
//             }
//         }
   
//     }
//     post { 
//         always { 
//             echo 'I will always say Hello again!'
//             deleteDir()
//         }
//         success { 
//             echo 'I will run when pipeline is success'
//         }
//         failure { 
//             echo 'I will run when pipeline is failure'
//         }
//     }
// }