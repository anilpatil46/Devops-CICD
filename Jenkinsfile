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
                    kubeconfig(caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJYnl1aHdUbkdiVjR3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpBNU1EZ3dOREkxTkRSYUZ3MHpNekE1TURVd05ETXdORFJhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURyT2xlV0Y3NVZpYkg3WkJRb1BvbThCRmljU1hjbUxvaVJlNGdIdVV5SmhCQjU4dzVrTEtzeTQ1UXkKblBzMnJLQ1U4T3VGV3ZPb0hwOEtQS0xaNWF0WXBGUnhSWS9aZ0VIVWozVEVXSFdsbTV6SXhiTjByWDlLM3haYgp2elRtREpMU3kySTVKMVk1OVNoRlVOK0lLaWNnOWdZQ0l1Qkx2cmlBU2Z4d1BTc0JNVGwvZVlSZkp1Wm1JeHVOCnRWSFdneVVQZDFTMG5SanVWUzZnb2RYbEdwM3YvRTZkMzRpekdMbk5BNXppZjhBbWluRUtoazZKYlpWMzFVcXEKeSt5eHlMOTNjaGpZcHhRZTFjSEFRenlwbGZOd3QyT2lhbHlrTTljWWRtOXd0bmZRRDFzMy9PQTMyT0UwVk4vegplRlM5Qlc2S2N1ZzM3aHA2d2I5b0xaa0wwc0tSQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJUalI2eDVtcHpTcVN1QXlsU3p2N1VYRjlIcHhUQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRRGlmcGYzRmZHMwpRWEg2bk90eEhPQWQ0OGJwNmtOTWpibXhHcnlZSFpRd2Q4NWNVOHVtRSs5N3k1V0pYV3pjR0pzaEZIakZUV2pCCjFPcG8vWE52MmhlbFZHTEVqbWZ6Yk9sZW80TVFOcHJYUFo0QW9QTG4rWFBFTEhIK2E5aE96bTMrY2haMnp1QmUKRjVmZXQ1aXVLVXVBRXBuZ2RJRFdmSEFMQW0wdGU1a2NaSlpVOTUxVkFQNXNualRRUU9FSjV4L2gxWkZZS0VMSgpLbGkzUHdlVjlJdk9jWCsreS9RaXpYS1l2U2tnaUNGRjlUMXNnM0RLVEdndFJPYkY0cVYwR3hhMzF2VFhHTkFOCndGZytqOFIzaWlIL0diVnFrRFpYZUlwcmxhSVFhUUZkeDV3aENNQmhSSlFMMk42c3ZjTGNMbDd3SDUyWWhlWWYKODFsTUZZcEVRVUsxCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K', credentialsId: 'k8_configfile_cred', serverUrl: 'https://192.168.0.110:6443') {
                    sh "kubectl apply -f deploymentservice.yaml"
}
                }
            }
        }
        
    }
}