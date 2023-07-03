pipeline {
    agent {label 'master'}
    def namespace = "test"
    def project = "shovel"
    def imageTag = new Date().format('yyyyMMddHHmm')
    def releaseImage = "registry.cn-hangzhou.aliyuncs.com/shovel/shovel-kh:${imageTag}"
    def packageImage = "registry.cn-hangzhou.aliyuncs.com/shovel-build/shovel-kh:${imageTag}"
    def branch = params.BRANCH

    stages {
      stage('Init') {
      steps {
        cleanWs()
        checkout scm
        dir(env.WORKSPACE) {
          sh "pwd"
        }
      }
    }
      stage('Package') {
        steps {
          dir("${env.WORKSPACE}") {
           sh """
            echo "Package start"
            make build-package-image package-image=${packageImage}
            make package package-image=${packageImage}
           """
          }
        }
     }
    stage('Release') {
        steps {
          dir("${env.WORKSPACE}") {
           sh """
            make build-release-image release-image=${releaseImage}
           """
          }
        }
     }

     stage('Push release image to registry') {
        steps {
          dir("${env.WORKSPACE}") {
           sh """
            echo "docker push ${releaseImage}"
           """
          }
        }
     }

      stage('Deploy') {
        steps {
          dir("${env.WORKSPACE}") {
           sh """
            echo "kubectl deploy"
           """
          }
        }
     }
}
}
