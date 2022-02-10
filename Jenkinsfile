def img
pipeline {
    environment {
        registry = "chet99/python-jenkins"
        registryCredential = "docker-hub-login"
        dockerImage = ""
    }
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/ChetanLaxane873/todoapp.git'
            }
        }
        
        stage("Stopping existing container") {
            steps{
                script{
                    sh returnStatus: true, script: 'docker stop $(docker ps -a -q)'
                    sh returnStatus: true, script: "docker rm ${JOB_NAME}"
                    sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry}) --force'    
                }
            }
            
        }
        
        stage('Build') {
            steps {
                script {
                    img = registry + ":${env.BUILD_ID}"
                    println("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }
        
        stage('Test') {
            steps {
                script{
                    sh "docker run -d --name ${JOB_NAME} -p 5000:5000 -t ${img}"
                }
            }
        }
        
        stage("Push to DockerHub") {
            steps {
                script { 
                    docker.withRegistry("https://registry.hub.docker.com", registryCredential) {
                        dockerImage.push()
                    }
                    
                }
            }
            
        }
        
        stage("Run in staging"){
            steps {
                script {
                    def stopcontainer = "docker stop ${JOB_NAME}"
                    def delcontName = "docker rm ${JOB_NAME}"
                    def delimage = "docker image prune -a --force"
                    def drun = "docker run -d --name ${JOB_NAME} -p 5001:5000 ${img}"
                    sshagent(['e19985e4-3aaa-4f5b-bdb8-c5abd965642a']) {
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no jenkins@172.31.29.91 ${stopcontainer}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no jenkins@172.31.29.91 ${delcontName}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no jenkins@172.31.29.91 ${delimage}"
                    
                        sh "ssh -o StrictHostKeyChecking=no jenkins@172.31.29.91 ${drun}"
                    } 
                }
            }
        }
    }
}
