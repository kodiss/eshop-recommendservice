/*
<< 변수 >> 치환 필요

<< ECR URI >>      => ex) 123456789012.dkr.ecr.us-east-1.amazonaws.com
<< SERVICE NAME >> => ex) eshop-backend
<< TAG >>          => ex) latest

<< ECR URI >>/<< SERVICE NAME >>:<< TAG >> => ex) 123456789012.dkr.ecr.us-east-1.amazonaws.com/eshop-backend:latest
*/
pipeline {
  agent {
    kubernetes {
      yaml """
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
"""
    }
  }
  stages {
    
    stage('Approval') {
      when {
        branch 'main'
      }
      steps {
        script {
          def plan = 'recommendservice CI'
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
        container(name: 'kaniko', shell: '/busybox/sh') {
          
          withCredentials([
          usernamePassword
            (credentialsId: 'github', 
             usernameVariable: 'USERNAME', 
             passwordVariable: 'GIT_TOKEN'
            )
          ])

          {
            sh '''#!/busybox/sh
            /kaniko/executor \\
            --git branch=main \\
            --context=git://$USERNAME:$GIT_TOKEN@github.com/$USERNAME/eshop-recommendservice.git \\
            --destination=100462131013.dkr.ecr.us-east-1.amazonaws.com/eshop-recommendservice:latest
            '''
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