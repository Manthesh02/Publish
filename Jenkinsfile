pipeline {
    agent any

    parameters {
        string(defaultValue: '1.0.0', description: 'Version number for the build', name: 'VERSION')
    }

    environment {
        GIT_USER_EMAIL = 'vmanthesh20@gmail.com'
        GIT_USER = 'Manthesh02'
        DOCKERHUB_CREDENTIALS = 'dockerhub'
    }

    tools {
        maven 'Maven'
    }

    stages {
        stage('Git Configuration') {
            steps {
                sh "git config --global user.email '${env.GIT_USER_EMAIL}'"
                sh "git config --global user.name '${env.GIT_USER}'"
            }
        }

        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'Github', url: 'git@github.com:Manthesh02/Publish.git']])
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Tag and Push') {
            steps {
                sh "git tag -a ${params.VERSION} -m 'Version ${params.VERSION}'"
                sh 'git push origin --tags'
            }
        }

        stage('Build Node JS Docker Image') {
            steps {
                script {
                    sh 'docker build -t manthesh/java-app-1.0 .'
                }
            }
        }

        stage('Deploy Docker Image to DockerHub') {
            steps {
                script {
                    withCredentials([string(credentialsId: env.DOCKERHUB_CREDENTIALS, variable: 'DOCKERHUB_PASSWORD')]) {
                        sh "docker login -u manthesh -p '${DOCKERHUB_PASSWORD}'"
                        sh 'docker push manthesh/java-app-1.0'
                    }
                }
            }
        }

        stage('Deploy to k3s') {
            steps {
                script {
                    sh '/usr/local/bin/kubectl apply -f /var/lib/jenkins/workspace/publish/app.yaml --kubeconfig /etc/rancher/k3s/k3s.yaml'
                }
            }
        }
    }
}
