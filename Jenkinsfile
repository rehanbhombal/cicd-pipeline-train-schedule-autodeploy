pipeline {
    agent any
    environment {
        // Replace with your Docker Hub username/repo
        DOCKER_IMAGE_NAME = "rehan2019/train-schedule"
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Running build automation'
                // Use full path to gradlew.bat to ensure Windows cmd works
                bat 'C:\\Windows\\System32\\cmd.exe /c gradlew.bat build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip', allowEmptyArchive: true
            }
        }

        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo 'Building Docker image'
                bat "docker build -t %DOCKER_IMAGE_NAME% ."
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo 'Pushing Docker image to Docker Hub'
                withCredentials([usernamePassword(credentialsId: 'docker_hub_login', 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                    bat "docker push %DOCKER_IMAGE_NAME%:%BUILD_NUMBER%"
                    bat "docker push %DOCKER_IMAGE_NAME%:latest"
                }
            }
        }

        stage('Canary Deploy') {
            when {
                branch 'master'
            }
            environment {
                CANARY_REPLICAS = 1
            }
            steps {
                echo 'Deploying Canary to Kubernetes'
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            environment {
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                echo 'Deploying to Production'
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}