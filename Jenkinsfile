#! groovy

node {
    agent {label 'cpp-slave'}
    def namespace = "test"
    def project = "shovel"
    def imageTag = new Date().format('yyyyMMddHHmm')
    def releaseImage = "registry.cn-hangzhou.aliyuncs.com/shovel/shovel-kh:${imageTag}"
    def packageImage = "registry.cn-hangzhou.aliyuncs.com/shovel-build/shovel-kh:${imageTag}"
    def branch = params.BRANCH

    try {
        stage('Clone target repo') {
             checkout([$class: 'GitSCM', branches: [[name: branch]],
             doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [],
             userRemoteConfigs: [[credentialsId: 'deploy', url: 'https://github.com/ChinaLHR/shovel-kubernetes-helm-devops.git']]])
        }

        stage('Build package') {
            sh "make build-package-image package-image=${packageImage}"
            sh "make package package-image=${packageImage}"
        }

        stage('Build release') {
            sh "make build-release-image release-image=${releaseImage}"
        }

        stage('Push release image to registry') {
            withDockerRegistry(credentialsId: 'docker-user', url: 'https://registry.cn-hangzhou.aliyuncs.com') {
                sh "docker push ${releaseImage}"
            }
        }

        stage('Deploy') {
            sh """
               kubectl config use-context dev
               helm -n ${namespace} upgrade ${project} chart \
                    -f chart/values.yaml \
                    --set-string image.tag=${imageTag} \
                    --wait --install
                """
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
    } finally {
            sh """
               docker image rm ${releaseImage} || true
               docker image rm ${packageImage} || true
            """
            deleteDir()

    }
}
