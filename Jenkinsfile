def getVersion(){
    def commitHash =  sh returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
pipeline{
    agent any

    options{
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
    }

    environment{
        GIT_COMMIT_HASH = getVersion()
        project = "$project"
        scannerHome = "${JENKINS_HOME}"

    }

    stages{
        stage("Code checkout"){
            steps{
                echo "[INFO] Checking out latest code from git"
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Microservices-starter/wishlist-microservice.git'                
            }
        }

        stage("Sonar analysis"){
            steps{
                echo "[INFO] Performing analysis with Sonarqube"
                script{
                    withSonarQubeEnv(credentialsId: 'sonartoken'){
                        sh "sudo ${scannerHome}/sonar-scanner/bin/sonar-scanner -Dsonar.projectKey=$project"
                    }
                }
            }

        }

        stage("Quality Gates"){
            steps{
                echo "[INFO] Verifying Quality gates"
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Docker build"){
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
