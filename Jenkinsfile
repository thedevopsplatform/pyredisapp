pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "thedevopsplatform/pyredisapp"
        CANARY_REPLICAS = 2
    }
    stages {
        stage("Build"){
            steps {
                echo 'Running build Automation'
            }
        }
        stage("Test"){
            steps {
                echo 'Running Test Automation'
            }
        }
        stage("Build Docker Image"){
            when {
                branch 'master'
            }
            steps {
                sh "docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} ."
            }    
        }
        stage("Docker Push"){
            when {
                branch 'master'
            }
            steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_CRED', variable: 'DOCKER_HUB_CRED')]) {
                    sh "docker login -u thedevopsplatform -p ${DOCKER_HUB_CRED}"
        }
                sh "docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                //sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                //now sent latest build
            }
        }    

        stage("Deploy To Staging"){
            when {
                branch 'master'
            }
            environment{
                CANARY_REPLICAS = 1    
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'staging-deployment.yml',
                    enableConfigSubstitution: true
                )
                // sh "kubectl apply -f staging-deployment.yml"
                // sh "kubectl apply -f ingress.yml"
            }
        } 
        stage("Deploy To Production"){
            when {
                branch 'master'
            }
            environment{
                CANARY_REPLICAS = 0
                PROD_REPLICAS = 2    
            }            
            steps {
                input 'Deploy to Prod?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'staging-deployment.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'prod-deployment.yml',
                    enableConfigSubstitution: true
                )
                // sh "kubectl apply -f staging-deployment.yml"
                // sh "kubectl apply -f prod-deployment.yml"
            }
        }    
    }    
}