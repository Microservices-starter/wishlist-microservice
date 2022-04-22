def getVersion(){
    def commitHash =  sh returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}

pipeline{
    agent any

    options{
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '30'))
    }

    environment{
        GIT_COMMIT_HASH = getVersion()
    }

    stages{
        stage("Code Checkout"){
            steps{
                echo "[INFO] Checking out latest code from git"
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Microservices-starter/wishlist-microservice.git' 
            }
        }

        stage("Sonar Analysis"){
            steps{
                echo "[INFO] Sonar Analysis"
                sh 'sonar-scanner -X -Dsonar.projectKey=wishlist'
            }
        }

        stage("Docker Build"){
            steps{
                echo "[INFO] Building Docker images"
                sh 'docker build -t rajputmarch2020/wishlist:$GIT_COMMIT_HASH .'
            }
        }

        stage("Docker push"){
            steps{
                echo "[INFO] Pushing Docker images to Dockerhub"
                withCredentials([string(credentialsId: 'dockerhub', variable: 'password')]){
                    sh 'docker login -u rajputmarch2020 -p ${password} '

                }
                    sh 'docker push rajputmarch2020/wishlist:$GIT_COMMIT_HASH'
            }
        }

    }
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}