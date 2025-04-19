pipeline {
    agent any

    environment {
        PATH = "/usr/bin:$PATH"
        tag = "1.0"
        dockerHubUser = "shyla414" // replace with your actual DockerHub username
        containerName = "insure-me"
        httpPort = "8081"
    }

    stages {
        stage("Code Clone") {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']], // or master if your repo uses that
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/shailajabeeram/asiinsurance.git']]
                )
            }
        }

        stage("Maven Build") {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }

        stage("Build Docker Image") {
            steps {
                sh "docker build -t ${dockerHubUser}/${containerName}:${tag} ."
            }
        }

        stage("Push Image to DockerHub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerHubAccount',
                    usernameVariable: 'dockerUser',
                    passwordVariable: 'dockerPassword'
                )]) {
                    sh "echo $dockerPassword | docker login -u $dockerUser --password-stdin"
                    sh "docker push $dockerUser/$containerName:$tag"
                }
            }
        }

        stage("Docker Container Deployment") {
            steps {
                sh "docker rm -f $containerName || true"
                sh "docker pull $dockerHubUser/$containerName:$tag"
                sh "docker run -d -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
                echo "🚀 Application started on http://<your-ec2-ip>:${httpPort}"
            }
        }
    }
}

