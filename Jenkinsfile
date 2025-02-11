pipeline {
    agent {label 'master'}
    environment {
        namespace = "test"
        project = "shovel"
        imageTag = new Date().format('yyyyMMddHHmm')
        releaseImage = "testk/shovel/shovel-kh:${imageTag}"
        packageImage = "testk/shovel-build/shovel-kh:${imageTag}"
    }

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
