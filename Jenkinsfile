pipeline {
  agent any
  environment {
    WORKSPACE = "${env.WORKSPACE}"
  }
  tools {
    maven 'localMaven'
    jdk 'localJdk'
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
      post {
        success {
          echo ' now Archiving '
          archiveArtifacts artifacts: '**/*.war'
        }
      }
    }
    stage('SonarQube Scan') {
      steps {
        sh """mvn sonar:sonar \
  -Dsonar.host.url=http://35.165.132.16:9000 \
  -Dsonar.login=5b29069c381022d700f83ca3402ed8c78bd70c0d"""
      }
    }
    stage('Upload to Artifactory') {
      steps {
        sh "mvn clean deploy -DskipTests"
      }

    }
    stage('Deploy to us-west-2') {
      environment {
        HOSTS = "us-west-2"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }

    }
    stage('Approval') {
      steps {
        input('Do you want to proceed?')
      }
    }
    stage('Deploy to ap-south-1') {
      environment {
        HOSTS = "ap-south-1"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
    }
  }

}
