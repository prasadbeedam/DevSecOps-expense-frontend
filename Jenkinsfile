pipeline {
    agent {
        label 'AGENT-1'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

    environment {
        appVersion = ''
        region = "us-east-1"
        account_id = "202533543549"
        ECR_REPO = "${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend"
    }

    stages {
        stage('Read Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Application version: ${appVersion}"
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                sh '''
                    echo "Running npm audit for vulnerabilities..."
                    npm install --silent
                    npm audit --audit-level=high || (echo "High severity vulnerabilities found!" && exit 1)
                '''
            }
        }

        stage('Build Artifacts') {
            steps {
                sh """
                    zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                    ls -ltr
                """
            }
        }

        stage('Docker Build & Scan') {
            steps {
                script {
                    sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

                     docker build -t ${ECR_REPO}:${appVersion} .

                     # Scan Docker image using trivy container
                     docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --exit-code 1 --severity HIGH,CRITICAL ${ECR_REPO}:${appVersion}

                      docker push ${ECR_REPO}:${appVersion}
                    """
                }
            }
        }

        stage('Optional: Sign Docker Image') {
            steps {
                script {
                    // Ensure cosign is installed on the agent
                    sh """
                        cosign sign ${ECR_REPO}:${appVersion}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    aws eks update-kubeconfig --region ${region} --name expense-dev
                    kubectl apply -f manifest.yaml
                """
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "frontend-${appVersion}.zip", fingerprint: true
        }
        failure {
            echo "Pipeline failed! Please check the logs."
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
//     environment{
//         def appVersion = '' //variable declaration
//         // nexusUrl = 'nexus.daws78s.online:8081'
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
//         stage('Build'){
//             steps{
//                 sh """
//                 zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
//                 ls -ltr
//                 """
//             }
//         }

//         stage('Docker build'){
//             steps{
//                 sh """
//                     aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

//                     docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend:${appVersion} .

//                     docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend:${appVersion}
//                 """
//             }
//         }

//         stage('deploy') {
//             steps {
//                 sh """
//                 aws eks update-kubeconfig --region us-east-1 --name expense-dev
//                 kubectl apply -f manifest.yaml
//                 """
//             }
//         }
        
//     }
// }
