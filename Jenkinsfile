pipeline {
  agent {
    kubernetes {
      yaml """
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.6.0-debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
"""
    }
  }
  environment {
    SCP_CREDS = credentials('scpCredentials')
    DOCKER_REGISTRY = "eshopregistry-dklbuzel.scr.kr-west.scp-in.com"
  }
  stages {
    stage('Approval') {
      when {
        branch 'main'
      }
      steps {
        script {
          def plan = 'recommentservice CI'
          input message: "Do you want to build and push?",
              parameters: [text(name: 'Plan', description: 'Please review the work', defaultValue: plan)]
        }
      } 
    }
    stage('Build with Kaniko') {
      when {
        branch 'main'
      }
      steps {
        container('kaniko') {
          script{
            def dockerAuth = sh(returnStdout: true, script: "echo -n \"${SCP_CREDS_USR}:${SCP_CREDS_PSW}\" | base64").trim().replaceAll(/\n/, '')            
            sh """
              rm -rf /kaniko/.docker
              mkdir /kaniko/.docker
              echo '{\"auths\":{\"${DOCKER_REGISTRY}\":{\"auth\":\"$dockerAuth\"}}}' > /kaniko/.docker/config.json
              cat /kaniko/.docker/config.json
              /kaniko/executor \
                --git branch=master \
                --context=. \
                --destination=${DOCKER_REGISTRY}/eshop-recommentservice:latest
            """
          }
        }
      }
      post {
        success { 
          slackSend(channel: 'U0570DT5003', color: 'good', message: "(Job:${env.JOB_NAME}-Build Number : ${env.BUILD_NUMBER}) CI success from th1227.kim")
        }
        failure {
          slackSend(channel: 'U0570DT5003', color: 'danger', message: "(Job:${env.JOB_NAME}-Build Number : ${env.BUILD_NUMBER}) CI fail from th1227.kim")
        }
      }
    }
  }
}