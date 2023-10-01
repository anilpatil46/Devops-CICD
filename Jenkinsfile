pipeline {   
    agent any 
    tools{
        jdk 'jdk11'
        maven 'maven3'
        
    }
    environment {
        SCANNER_HOME= tool 'sonarqube-scanner'
        APP_NAME = "devops-cicd"
        RELEASE = "1.0.0"
        DOCKER_USER = "anilpatil46"
        DOCKER_PASS = 'docker_cred'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
       
    }

    stages {

        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }
        }

        stage('Git checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/anilpatil46/Devops-CICD.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Package') {
            steps {
                sh "mvn clean package"
                
            }
        }
        stage("Test Application"){
            steps {
                sh "mvn test"
            }

        }
        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: '27f4b246-1640-45d4-a92d-6e37933017d3') {
                    sh "mvn sonar:sonar"
                    }
                }
            }

        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: '27f4b246-1640-45d4-a92d-6e37933017d3'
                }
            }

        }
        
        stage("Build Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                }    
            }
        }

        stage("Trivy-Scan Docker Image") {
            steps {
                echo "************ Scanning Docker Image with Trivy ****************"
                sh 'trivy --severity CRITICAL image anilpatil46/devops-cicd'
            }

        }

        stage("Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

        }
        stage('Deploy to k8s'){
            steps{
                script{
                    kubernetesDeploy(configs: 'deploymentservice.yaml',kubeconfigId: 'k8_configfile_cred')
                }
            }
        }
        
    }
}