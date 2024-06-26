#!groovy

pipeline {
  agent none
  stages {
    stage('Nginx Install') {
      agent {
        docker {
          image 'nginx:latest'
        }
      }
      steps {
        sh 'echo hello_world'
      }
    }
    stage('Docker Build') {
      agent any
      steps {
        sh 'docker build -t nginx:latest .'
      }
    }
    stage('Docker Push') {
      agent any
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
          sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
	  sh "docker run -d -p 9889:80 --name myapp nginx:latest"
        }
      }
    }
    stage('send telegram') {
	agent any
	steps {
        withCredentials([string(credentialsId: 'botSecret', variable: 'TOKEN'), string(credentialsId: 'chatWebid', variable: 'CHAT_ID')]) {
        sh  ("""
            curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC *Branch*: ${env.GIT_BRANCH} *Build* : OK *Published* = YES'
        """)
        }
	}
     }
    stage('RUN scripts') {
      agent any
      steps {
	  sh 'bash -vx script_curl.sh'
          sh 'bash -vx script.sh'
      }
    }
    stage('Delete container') {
      agent any
      steps {
          sh 'docker rm -f myapp'
	  sh 'docker rmi nginx:latest'
      }
    }
  }
}
