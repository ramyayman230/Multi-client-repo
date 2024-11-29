pipeline {
    agent any 
    tools {
        maven 'Maven'
    }
    environment {
        dockerhub_cred = credentials('docker-cred')
        KUBECONFIG_CRED = credentials('kubeconfig')
    }
    
    stages {
        stage('Maven Build') {
            steps {
               sh 'mvn package'
            }
        }
        stage('Docker Build') {
            steps {
               sh '''#!/bin/bash
               docker build -t romeo23/client-${env.BRANCH_NAME}-jenkins:${BUILD_NUMBER} .
               '''
            }
        }
        stage('List Images') {
            steps {
               sh 'docker images'
            }
        }
        stage('DockerHub Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        sh '''#!/bin/bash
                        echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                        docker push romeo23/client-${env.BRANCH_NAME}-jenkins:${BUILD_NUMBER}
                        docker tag romeo23/client-${env.BRANCH_NAME}-jenkins:${BUILD_NUMBER} romeo23/client-${env.BRANCH_NAME}-jenkins:latest
                        docker push romeo23/client-${env.BRANCH_NAME}-jenkins:latest
                        '''
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''#!/bin/bash
                    kubectl apply -f client-deployment.yaml
                    kubectl apply -f client-service.yaml
                    '''
                }
            }
        }
    }
}
