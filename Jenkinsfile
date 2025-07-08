pipeline {

    agent any
    //escenarios -> escenario -> pasos
    environment{
        NPM_CONFIG_CACHE="${WORKSPACE}/.npm"
        dockerimagePrefix = "us-west1-docker.pkg.dev/lab-agibiz/docker-repository"
        registry = "https://us-west1-docker.pkg.dev"
        registryCredentials = 'gcp-registry'
    }
    stages{
        stage ("proceso de build y test") {
            agent {
                docker {
                    image 'node:22'
                    reuseNode true
                }
            }

            stages {
                stage("instalacion de dependencias"){
                    steps{
                        sh 'npm ci'
                    }
                }
                stage("ejecucion de pruebas"){
                    steps{
                        sh 'npm run test:cov'
                    }
                }
                stage("construccion de la aplicacion"){
                    steps{
                        sh 'npm run build'
                    }
                }

            }

        }
        stage ("build y push de imagen docker"){
            steps{
                script{
                    docker.withRegistry("${registry}", registryCredentials){
                        sh "docker build -t backend-nest-test-fgm ."
                        sh "docker tag backend-nest-test-fgm ${dockerimagePrefix}/backend-nest-test-fgm"
                        sh "docker tag backend-nest-test-fgm ${dockerimagePrefix}/backend-nest-test-fgm:${BUILD_NUMBER}"
                        sh "docker push ${dockerimagePrefix}/backend-nest-test-fgm"
                        sh "docker push ${dockerimagePrefix}/backend-nest-test-fgm:${BUILD_NUMBER}"
                    }
                }

            }
        }
        stage ("actualizacion de kubernetes"){
            agent {
                docker {
                    image 'alpine/k8s:1.30.2'
                    reuseNode true
                }
            }
            steps {
                withKubeConfig([credentialsId: 'gcp-kubeconfig']){
                    sh 'kubectl -n lab-fgm set image deployments/backend-nest-test-fgm backend-nest-test-fgm=${dockerimagePrefix}/backend-nest-test-fgm:${BUILD_NUMBER}'
                }
            }
        }
    }
}