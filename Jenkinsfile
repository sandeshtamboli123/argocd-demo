pipeline {
  agent {
    kubernetes {
      label 'jenkins-slave'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: dind
    image: docker:18.09-dind
    securityContext:
      privileged: true
  - name: docker
    env:
    - name: DOCKER_HOST
      value: 127.0.0.1
    image: docker
    command:
    - cat
    tty: true
  - name: tools
    image: nekottyo/kustomize-kubeval
    command:
    - cat
    tty: true  
"""
    }
  }
  stages {

    stage('Image Build') {
      environment {
        DOCKERHUB_CREDS = credentials('dockerHub')
      }
      steps {
        container('docker') {
          sh "docker build -t mynamesandesh/argocd-demo:${env.GIT_COMMIT} ."
          sh "docker login --username $DOCKERHUB_CREDS_USR --password $DOCKERHUB_CREDS_PSW" 
          sh "docker push mynamesandesh/argocd-demo:${env.GIT_COMMIT}"
        }
      }
    }

    stage('Deploy to qa') {
      environment {
        GIT_CREDS = credentials('new-github')
      }
      steps {
        container('tools') {
            sh "git clone https://.Data.data$GIT_CREDS_USR:.Data.data$GIT_CREDS_PSW@github.com/sandeshtamboli123/argocd-demo-deploy.git"
            sh "git config --global user.email sandeshtamboli123@gmail.com"
            sh "git config --global user.name sandesh"

            dir("argocd-demo-deploy") {
            sh "cd ./overlays/qa && kustomize edit set image my-app=mynamesandesh/argocd-demo:${env.GIT_COMMIT}"
            sh "git commit -am 'Update app image tag to ${env.GIT_COMMIT}'"
            sh "git push"
            }    
        }
     }
   }
   stage('Deploy to prod') {
      environment {
        GIT_CREDS = credentials('new-github')
      }
      steps {
        container('tools') {
            sh "git clone https://$GIT_CREDS_USR:$GIT_CREDS_PSW@github.com/sandeshtamboli123/argocd-demo-deploy.git"
            sh "git config --global user.email sandeshtamboli123@gmail.com"
            sh "git config --global user.name sandesh"

            dir("argocd-demo-deploy") {
            sh "cd ./overlays/prod && kustomize edit set image my-app=mynamesandesh/argocd-demo:${env.GIT_COMMIT}"
            sh "git commit -am 'Update app image tag to ${env.GIT_COMMIT}'"
            sh "git push"
            }    
        }
      }
   }
}
}
