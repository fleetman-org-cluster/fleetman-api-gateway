pipeline {
  agent any

  tools {
        // Define the Maven installation
        maven 'maven'
    }

  environment {
    // You must set the following environment variables
    // ORGANIZATION_NAME
    // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)
    PATH = "$PATH:/usr/local/bin"
    ORGANIZATION_NAME = "fleetman-org-cluster"
    SERVICE_NAME = "fleetman-api-gateway"
    REPOSITORY_TAG = "${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
  }

  stages {
    stage('Preparation') {
      steps {
        cleanWs()
        git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
        checkout scm
      }
    }
    stage('Install Gettext') {
        steps {
            //sh 'apt-get update -y && apt-get install -y gettext' on linux environment debian
            // sh 'brew update && brew install gettext' // mac os, you should install homebrew on your mac
          sh 'xcode-select --update' // Install Xcode command-line tools (if not already installed)
                sh 'curl -O https://ftp.gnu.org/gnu/gettext/gettext-0.21.tar.gz'
                sh 'tar -xzf gettext-0.21.tar.gz'
                dir('gettext-0.21') {
                    sh './configure'
                    sh 'make'
                    sh 'sudo make install'
                }
        }
    }
    stage('Build') {
      steps {
        sh '''mvn clean package'''
      }
    }

    stage('Build and Push Image') {
      steps {
        sh 'docker image build -t ${REPOSITORY_TAG} .'
      }
    }

    stage('Deploy to Cluster') {
      steps {
        sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
      }
    }
  }
}
